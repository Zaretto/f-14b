# F-14 Tomcat Electrical System Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27

## Overview

The F-14 Tomcat electrical system model provides a simplified 28-volt DC power distribution network. While the real F-14 has both AC and DC systems with multiple generators, this simulation focuses on the essential electrical power requirements for avionics, instruments, and emergency systems.

## System Architecture

### Power Sources

Located in: `Systems/f-14b-electrical.xml`

#### Battery 1

```xml
<supplier>
  <name>Battery 1</name>
  <prop>/systems/electrical/suppliers/battery[0]</prop>
  <kind>battery</kind>
  <volts>28</volts>
  <amps>60</amps>
</supplier>
```

Primary battery characteristics:
```
Voltage: 28 VDC nominal
Capacity: 60 amp-hours (estimated)
Type: Lead-acid (real aircraft uses NiCd)
Purpose: Emergency backup power, engine starting assist

Powers:
  - Essential instruments during electrical failure
  - Emergency lighting
  - Essential avionics (limited duration)
```

#### Alternator 1 (Engine-Driven Generator)

```xml
<supplier>
  <name>Alternator 1</name>
  <prop>/systems/electrical/suppliers/alternator[0]</prop>
  <kind>alternator</kind>
  <rpm-source>/engines/engine[0]/rpm</rpm-source>
  <volts>28</volts>
  <amps>60</amps>
</supplier>
```

Primary generator characteristics:
```
Drive: Engine 0 (left engine) accessory gearbox
Voltage: 28 VDC
Current: 60 amps continuous
Activation: Automatic when engine RPM > threshold

Real Aircraft:
  - Two engine-driven generators (400Hz AC)
  - Multiple transformer-rectifiers for DC
  - APU generator for ground/emergency power

Model Simplification:
  - Single DC generator (engine 0)
  - Adequate for essential system modeling
```

#### External Power

```xml
<supplier>
  <name>External 1</name>
  <prop>/systems/electrical/suppliers/external[0]</prop>
  <kind>external</kind>
  <volts>0</volts>
  <amps>0</amps>
</supplier>
```

Ground power unit (GPU) connection:
```
Voltage: 28 VDC (when connected)
Current: Unlimited (ground cart)
Use: Ground operations, maintenance, pre-flight
Default: Disconnected (0 volts)

To enable:
  Set property: systems/electrical/suppliers/external[0] = 28
```

### Power Distribution

#### Master Bus

The primary distribution point:

```xml
<bus>
  <name>Master Bus</name>
  <prop>/systems/electrical/outputs/bus[0]</prop>
</bus>
```

Supplies power to:
- All avionics directly connected to bus
- Circuit breaker-protected outputs
- Essential instruments
- Lighting
- Utility systems

### Power Source Priority

```
Connection logic:
  1. Alternator 1 → Master Bus (if engine running)
  2. External 1 → Master Bus (if connected)
  3. Battery 1 → Master Bus (if battery master switch ON)

Normal Operation:
  - Engine running: Alternator powers all systems
  - Battery charges from alternator
  - External power disconnected

Ground Operations:
  - External power connected: Powers all systems
  - Battery charging possible
  - Engine not required

Emergency:
  - Engine failed, no external power
  - Battery powers essential loads only
  - Limited duration (~30 minutes at reduced load)
```

## Electrical Loads

### Avionics (Directly Connected to Master Bus)

These systems have no circuit breakers (always powered when bus energized):

```
- Direction Gyro (DG) - Required for BDHI
- Avionics cooling fan
- GPS/MFD
- GPS navigation
- HSI (Horizontal Situation Indicator)
- NAV[0] receiver
- DME (Distance Measuring Equipment)
- Audio panel [0]
- NAV[1] receiver
- Audio panel [1]
- Transponder
- Autopilot computer
- ADF (Automatic Direction Finder)
- MK-VIII GPWS (Ground Proximity Warning)
- TACAN (Tactical Air Navigation)
```

### Circuit Breaker-Protected Outputs

#### Starter 1 Power

```xml
<output>
  <name>Starter 1 Power</name>
  <prop>/systems/electrical/outputs/starter[0]</prop>
</output>
```

Connected to Master Bus via switch:
```
Switch: /controls/engines/engine[0]/starter
Use: Engine starting (pneumatic starter assist)
Load: High current (>100 amps during start)
Duration: Brief (5-10 seconds per start)
```

#### Cabin Lights

```xml
<connector>
  <input>Master Bus</input>
  <output>Cabin Lights Power</output>
  <switch>
    <prop>/controls/circuit-breakers/cabin-lights-pwr</prop>
  </switch>
</connector>
```

Protected by circuit breaker, controlled by switch.

#### Instrument Power

```
Switch: /controls/circuit-breakers/instr-ignition-switch
Powers: Primary flight instruments, ignition system
Critical: Essential for flight
```

#### Fuel Pump Power

```
Switch: /controls/engines/engine[0]/fuel-pump
Powers: Electric fuel boost pumps
Use: Engine start, high-altitude flight, emergencies
```

#### Landing Light

```
Switch: /controls/switches/landing-light
Powers: Forward landing/taxi lights
Use: Night operations, low visibility
```

#### Beacon

```
Switch: /controls/switches/flashing-beacon
Powers: Rotating anti-collision beacon
Use: Aircraft visibility, all operations
```

#### Flaps

```
Circuit Breaker: /controls/circuit-breakers/flaps
Powers: Flap position indicators, controls
Note: Flaps hydraulically actuated, electrical for control only
```

#### Turn Coordinator

```
Circuit Breaker: /controls/circuit-breakers/turn-coordinator
Powers: Turn and bank indicator
Backup: Attitude indicator independent
```

#### Navigation Lights (Map Lights)

```
Switch: /controls/switches/nav-lights
Powers: Wing tip lights, tail light, formation lights
Use: Night flight, visibility
```

#### Instrument Lights

```
Circuit Breaker: /controls/circuit-breakers/instrument-lights
Powers: Cockpit panel lighting, flood lights
Critical: Night operations
```

#### Strobe Lights

```
Switch: /controls/switches/strobe-lights
Powers: High-intensity anti-collision strobes
Use: Enhanced visibility
```

#### Taxi Lights

```
Switch: /controls/switches/taxi-lights
Powers: Nose wheel taxi light
Use: Ground operations, taxiing
```

#### Pitot Heat

```
Switch: /controls/switches/pitot-heat
Powers: Pitot tube anti-ice heating element
Critical: Icing conditions, prevents airspeed loss
Use: Activate before flight in visible moisture
```

## Real F-14 Electrical System

For comparison, the actual aircraft has a more complex system:

### Real System Architecture

```
AC Power (400 Hz):
  - Left generator (L GEN): 40 KVA, engine 0 driven
  - Right generator (R GEN): 40 KVA, engine 1 driven
  - APU generator: 40 KVA, APU driven (ground/emergency)

AC Distribution:
  - Left AC bus
  - Right AC bus
  - Essential AC bus (critical systems)
  - Transfer logic (bus tie)

DC Power:
  - Transformer-Rectifier Units (TRUs) convert AC to DC
  - Left DC bus: 28 VDC, 200 amps
  - Right DC bus: 28 VDC, 200 amps
  - Essential DC bus: 28 VDC (battery backup)
  - Batteries: Two NiCd batteries (40 Ah each)

Emergency Power:
  - RAT (Ram Air Turbine) for hydraulics
  - Battery backup for essential AC/DC buses
  - Emergency generator from RAT (some variants)
```

### Model Simplifications

The FlightGear model simplifies this to:

```
Simplifications:
  1. Single DC bus (Master Bus) instead of multiple AC/DC buses
  2. Single generator (engine 0) instead of dual generators
  3. No APU generator modeled
  4. No transformer-rectifier units
  5. Simplified load modeling (no current draw calculation)
  6. No bus tie logic
  7. No emergency AC system

Reasons:
  - Adequate for essential systems
  - Reduces complexity
  - Maintains real-time performance
  - Focuses on flight-critical systems
```

## System Interactions

### With Engines

File: `Systems/f-14b-engines.xml` or `f-14b-engines-TF-30.xml`

Electrical system requires:
```
Engine running → Generator online → Electrical power

Engine start requires:
  - Battery or external power
  - Starter motor power (high current)
  - Ignition system power (instrument circuit)

Engine failure:
  - Generator offline
  - Battery backup (limited duration)
  - Emergency procedures required
```

### With Hydraulics

File: `Systems/f-14b-hydraulic.xml`

Emergency hydraulic pump requires:
```
Property: systems/electrics/ac-right-main-bus
Required: >5 volts for emergency pump operation

Failure:
  - No electrical power → No emergency hydraulic pump
  - Dual hydraulic failure becomes catastrophic
```

This creates critical interdependency: Electrical failure can cause hydraulic emergency.

### With CADC

File: `Systems/f-14b-cadc.xml`

AOA indexer requires electrical power:
```
Property: systems/electrics/ac-essential-bus1
Required: >0 volts for indexer operation

With power:
  - Indexer lights operate normally
  - AOA guidance available

Without power:
  - Indexer lights dark
  - Must estimate approach speed
  - Increased workload
```

### With Instruments

Many instruments require electrical power:
- Attitude gyros (essential bus)
- Navigation radios (Master Bus)
- Flight director (avionics bus)
- Radar (AC bus in real aircraft)

Electrical failure degrades situational awareness significantly.

## Key Properties (Property Tree)

### Supplier Voltages

```
systems/electrical/suppliers/battery[0]      (0-28 volts)
systems/electrical/suppliers/alternator[0]   (0-28 volts)
systems/electrical/suppliers/external[0]     (0-28 volts)
```

### Bus Voltages

```
systems/electrical/outputs/bus[0]            (0-28 volts) Master Bus
systems/electrics/ac-right-main-bus          (0-28 volts) AC equivalent
systems/electrics/ac-essential-bus1          (0-28 volts) Essential bus
```

Note: "AC" buses in properties are actually DC in this simplified model.

### Output Circuits

```
systems/electrical/outputs/starter[0]        (0-28 volts)
systems/electrical/outputs/cabin-lights      (0-28 volts)
systems/electrical/outputs/instr-ignition-switch  (0-28 volts)
systems/electrical/outputs/fuel-pump         (0-28 volts)
systems/electrical/outputs/landing-light     (0-28 volts)
systems/electrical/outputs/beacon            (0-28 volts)
systems/electrical/outputs/flaps             (0-28 volts)
systems/electrical/outputs/turn-coordinator  (0-28 volts)
systems/electrical/outputs/map-lights        (0-28 volts)
systems/electrical/outputs/instrument-lights (0-28 volts)
systems/electrical/outputs/strobe-lights     (0-28 volts)
systems/electrical/outputs/taxi-lights       (0-28 volts)
systems/electrical/outputs/pitot-heat        (0-28 volts)
systems/electrical/outputs/DG                (0-28 volts)
... (avionics outputs) ...
```

### Control Switches

```
controls/engines/engine[0]/master-alt        (0 or 1) Alternator master
controls/engines/engine[0]/master-bat        (0 or 1) Battery master
controls/engines/engine[0]/starter           (0 or 1) Starter switch
controls/circuit-breakers/...                (0 or 1) Various CBs
controls/switches/...                        (0 or 1) Various switches
```

## Operating Procedures

### Normal Start

```
1. External Power: CONNECT (if available)
   - Or rely on battery

2. Battery Master: ON
   - Energizes essential systems
   - Enables starter

3. Master Alternator: OFF (initially)
   - Prevents generator load during start

4. Engine Start:
   - Starter: ENGAGE (max 30 seconds)
   - Monitor engine instruments
   - Release starter at idle

5. Master Alternator: ON
   - Generator comes online (engine running)
   - Battery begins charging
   - External power: DISCONNECT

6. Check electrical system:
   - Generator online (>24 volts)
   - Battery charging
   - No warning lights
```

### In-Flight Normal

```
Monitor:
  - Generator output: 26-28 volts
  - No electrical warning lights
  - Ammeter: Small positive (battery charging)

Electrical Load Management:
  - All systems operational
  - Generator supplies entire load
  - Battery fully charged (standby)
```

### Electrical Failure (Generator)

```
Indications:
  - Generator warning light
  - Voltmeter: <24 volts (battery only)
  - Ammeter: Large negative (discharge)

Actions:
  1. Generator reset: ATTEMPT (cycle master alt switch)
  2. If unsuccessful:
     a. Master alternator: OFF (isolate failed generator)
     b. Shed non-essential loads:
        - Exterior lights (except minimum)
        - Avionics fan
        - Non-essential radios
        - Pitot heat (if not icing)
     c. Reduce electrical load to essential only
     d. Land as soon as practical

Battery Life:
  Essential loads only: ~30-45 minutes
  Full loads: ~10-15 minutes
```

### Total Electrical Failure (No Battery, No Generator)

```
Indications:
  - All electrical instruments dark
  - No radio
  - No lights
  - Cockpit instruments only: Standby (pneumatic)

Actions:
  1. Master battery: Check ON (may have tripped)
  2. Generator: Check (unlikely to recover)
  3. Fly aircraft with:
     - Standby attitude indicator (pneumatic)
     - Standby altimeter
     - Airspeed indicator
     - Magnetic compass
  4. Emergency procedures:
     - No radio (NORDO)
     - Visual signals for landing clearance
     - Day VFR only (no night/IFR)
  5. Land immediately

Difficulty:
  - No electrical flight controls (hydraulic still works)
  - Degraded situational awareness
  - No radio communication
  - Night/IMC flight impossible
```

## Emergency Procedures

### Alternator Failure

```
1. Generator warning light: ILLUMINATED
2. Master alternator: Cycle (OFF-ON) to reset
3. If warning persists:
   - Master alternator: OFF
   - Shed loads (see above)
   - Battery only - land ASAP
```

### Battery Failure

```
1. Indications:
   - Ammeter: Zero (no charge)
   - Voltmeter: 28V (generator okay)
   - Battery temp high (possible)

2. Actions:
   - Battery master: OFF (isolate)
   - Continue on generator power
   - No battery backup available
   - Land at nearest suitable field

Risk:
  - If generator fails, immediate electrical loss
  - No emergency power
```

### Circuit Breaker Trip

```
1. Identify tripped breaker (popped out)
2. Check affected system
3. If no smoke/fire/abnormality:
   - Reset breaker (push in) ONCE only
4. If trips again:
   - Leave open (fault in circuit)
   - Placard as INOP
   - Do not reset repeatedly (fire risk)
```

### Smoke in Cockpit (Electrical)

```
1. Source: Identify (smell, sight)
2. If electrical smoke:
   a. Battery master: OFF (immediately)
   b. Master alternator: OFF
   c. External power (if ground): DISCONNECT
3. Emergency ventilation: ESTABLISH
4. Smoke goggles: DON (if available)
5. If smoke clears:
   - Assess situation
   - Minimum electrical power (battery only if safe)
   - Land immediately
6. If smoke persists or fire evident:
   - EJECT (if airborne and uncontrollable)
   - EVACUATE (if on ground)
```

## Limitations of Model

### Not Modeled

```
1. Current draw (amperage) - No load calculation
2. Battery discharge rate - Infinite battery (no time limit)
3. Generator frequency (400 Hz) - Only DC modeled
4. Bus tie logic - Single bus only
5. Ground power interlock - Manual property set
6. APU generator - Not implemented
7. Circuit breaker thermal protection - Instant trip/reset
8. Wire resistance - No voltage drop
9. Dual generator load sharing - Single generator
10. Emergency AC systems - Not modeled
```

### Assumed Perfect Operation

```
- Generators always produce 28V (when online)
- Batteries never fail (unless modeling failure)
- Wiring never fails (except via CB trip)
- No shorts or ground faults
- Instant response (no relay delay)
```

### Purpose

Despite limitations, the model provides:
- Essential electrical power logic
- Realistic failure consequences
- Emergency procedure training
- Adequate fidelity for flight simulation

## Comparison with Real Aircraft

### Accurate Representations

- Battery backup capability
- Generator-driven by engine
- Circuit breaker protection
- Essential load priorities
- External power option

### Differences from Reality

| Real F-14 | Model |
|-----------|-------|
| Dual AC generators (40 KVA each) | Single DC generator |
| Multiple AC/DC buses | Single Master Bus |
| APU generator | Not modeled |
| Transformer-rectifiers | Direct DC |
| Current limiting | Not modeled |
| Load shedding automatic | Manual |

## References

1. **NAVAIR 01-F14AAD-1**: F-14A Aircraft Descriptive Data, Electrical System Section
2. **F-14 NATOPS**: Electrical system operating procedures
3. **FlightGear Electrical System Documentation**: Generic electrical system framework
4. **Implementation**: Richard Harrison, f-14b-electrical.xml

## Related Documentation

- **HYDRAULICS.md**: Emergency hydraulic pump electrical requirements
- **CADC.md**: AOA indexer electrical requirements
- **ENGINES.md**: Starter and ignition electrical power

---

*The electrical system model provides essential power distribution logic while maintaining simplicity for real-time simulation. It captures the critical dependencies and failure modes necessary for training and emergency procedures.*
