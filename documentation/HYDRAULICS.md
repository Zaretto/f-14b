# F-14 Tomcat Hydraulic System Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27
**Reference**: NAVAIR 01-F14AAD-1, Pages 2-68 to 2-76

## Overview

The F-14 Tomcat employs a dual hydraulic system architecture with bidirectional transfer capability and emergency backup. This sophisticated design ensures redundancy for critical flight control surfaces while optimizing system weight and complexity.

## System Architecture

### Two Independent Systems

1. **Flight System** (Right Engine Driven)
   - Primary pump: Engine 1 (right engine) N2-driven
   - Normal pressure: 3,000 PSI
   - Powers flight control surfaces

2. **Combined System** (Left Engine Driven)
   - Primary pump: Engine 0 (left engine) N2-driven
   - Normal pressure: 3,000 PSI
   - Powers utility systems and flight controls

### Bidirectional Transfer System

Unique feature allowing either system to supply the other:
- Activation pressure differential threshold
- Automatic engagement when one system pressure drops
- Transfer efficiency: 83% (modeled)
- Requires transfer pump switch activation

### Emergency Hydraulic System

- Electric motor-driven pump
- Automatic activation when both main systems drop below threshold
- Pressure: 2,001 PSI (in model)
- Powers essential flight controls
- AC-powered (requires electrical system)

## Physical System Description

### Hydraulic Pumps

Each main engine drives a dedicated hydraulic pump through the accessory gearbox:

```
Pump Drive:      Engine N2 shaft
Minimum N2:      18% (minimum windmilling for pressure)
Flow Rate:       Variable with engine speed
System Pressure: 3,000 PSI nominal
```

At 18% N2, pumps begin generating usable pressure (~130 PSI in model). This allows:
- Single engine windmilling at 18% N2: Sufficient for smooth flight controls
- Both engines at 11% N2 each: Barely adequate pressure

### Fluid Specifications

Standard MIL-H-83282 hydraulic fluid:
- Operating temperature range: -65°F to +275°F
- Fire-resistant formulation
- Compatible with system seals and materials

### System Capacity

```
Flight System:     Approximately 7.5 gallons
Combined System:   Approximately 7.5 gallons
Total:            ~15 gallons
```

## Implementation Details

### Flight System Channel

Located in: `f-14b-hydraulic.xml`, lines 15-103

#### Pump Pressure Generation

```xml
<scheduled_gain name="systems/hydraulics/flight-system-pump-psi">
  Input: Engine[1] N2

  Lookup Table:
    N2 = 0%:   PSI = 0
    N2 = 18%:  PSI = 130 (windmill minimum)

  Clipped to normal-pressure (3000 PSI)
```

This models the direct relationship between engine speed and hydraulic pressure.

#### Pressure Switch Logic

```
Flight system pressure OK when:
  system-psi ≥ 1800 PSI

Output: Binary flag (0 or 1)
```

Used for cockpit indicators and system logic.

#### Emergency Pump Activation

Automatic emergency pump logic:

```
Activates when:
  - Flight system PSI < 1100 (auto-activation threshold)
  - Combined system PSI < 1100
  - Emergency pump not failed
  - AC electrical power available (ac-right-main-bus > 5V)

Deactivates when:
  - Both systems recover above 1400 PSI (auto-deactivation)
  - OR emergency pump fails
```

Emergency pump provides 2,001 PSI to both systems.

#### Bidirectional Transfer (Combined → Flight)

```
Transfer occurs when:
  - Transfer pump switch ON
  - Combined PSI ≥ 1800 (bidi-activation threshold)
  - Flight pump PSI ≤ 1800
  - Both systems ≥ 800 PSI (cutoff-min-pressure)

Transfer amount:
  83% of Combined system pressure difference
  Capped at 2000 PSI transfer limit
```

The 83% factor represents line losses and pump efficiency.

#### Hydraulic Bleeding (Load)

Flight control actuation creates pressure drop:

```
Bleed sources:
  - Elevator movement (primary consumer)
  - Calculated from cmd vs. actual position

Load calculation:
  Base load × Emergency pump factor

  Emergency pump on: Factor = 100 (higher load)
  Emergency pump off: Factor = 30 (lower load)

  Capped: 0 to 300 PSI bleed maximum
```

This models the reduced flow capacity of the emergency pump.

### Combined System Channel

Located in: `f-14b-hydraulic.xml`, lines 105-171

Mirrors flight system architecture:

```
Pump: Engine[0] N2-driven
Pressure: Same curve as flight system
Transfer: Flight → Combined (83% efficiency)
Emergency: Shared emergency pump
```

#### Transfer Direction (Flight → Combined)

Symmetric with Combined → Flight transfer:

```
Activates when:
  - Transfer pump switch ON
  - Flight PSI ≥ 1800
  - Combined pump PSI ≤ 1800
  - Both systems ≥ 800 PSI
```

### System Pressure Summation

Final system pressure for each:

```
Flight System PSI =
  + Flight pump PSI
  + Bidi transfer from Combined PSI
  + Emergency transfer PSI
  - Control surface bleed (load)

Combined System PSI =
  + Combined pump PSI
  + Bidi transfer from Flight PSI
  + Emergency transfer PSI
  (no load modeled - utility consumers not simulated in detail)
```

## Operational Characteristics

### Normal Operations (Both Engines Running)

```
Flight System:    3000 PSI from Engine 1
Combined System:  3000 PSI from Engine 0
Transfer Pumps:   Inactive (unnecessary)
Emergency Pump:   Inactive
```

All flight controls operate normally with full authority and speed.

### Single Engine Failure Scenarios

#### Left Engine (Engine 0) Failure

```
Flight System:    3000 PSI from Engine 1 (unaffected)
Combined System:  Initially drops toward 0 PSI
Transfer:         Automatically engages
Result:           Combined receives ~2490 PSI from Flight
                  (83% of 3000 PSI)
```

Flight controls remain fully operational. Some utility systems may be degraded.

#### Right Engine (Engine 1) Failure

```
Flight System:    Initially drops toward 0 PSI
Combined System:  3000 PSI from Engine 0 (unaffected)
Transfer:         Automatically engages
Result:           Flight receives ~2490 PSI from Combined
```

Flight controls remain fully operational through bidirectional transfer.

### Both Engines Windmilling

```
Scenario: Dual flameout at altitude

Engine 0 N2:  18% (windmill)
Engine 1 N2:  18% (windmill)

Flight System:    130 PSI from Engine 1
Combined System:  130 PSI from Engine 0
Emergency Pump:   ACTIVATES (both systems < 1100 PSI)

Result:
  Flight System:    2001 PSI emergency
  Combined System:  2001 PSI emergency
```

Flight controls remain operational but with:
- Reduced actuator speed
- Increased control system bleed (modeled via factor)
- Adequate for safe landing

### Emergency Pump Only

```
Scenario: Both engines failed, AC power available

Emergency Pump: 2001 PSI to both systems
Control Authority: Reduced but adequate
Electrical Load: Significant draw on AC bus
```

Critical for:
- Glide approach and landing
- Wave-off with engine restart
- Emergency flight control

### System Pressure Loss Scenarios

#### Gradual Pressure Loss

If system pressure drops below minimum (< 800 PSI):
- Transfer system cuts off (bidi-cutoff-min-pressure)
- Emergency pump may not sustain both systems
- Flight controls become degraded or inoperative

#### Catastrophic Failure

```
Pump failure flag set (systems/hydraulics/*-pump-failed = 1):
  - Pump output forced to 0 PSI
  - System relies entirely on transfer or emergency
  - If transfer system failed, control loss in that system
```

## Flight Control Dependencies

### Hydraulic-Dependent Systems

The following require adequate hydraulic pressure:

1. **Primary Flight Controls**
   - Stabilator (pitch control)
   - Spoilers (roll control and DLC)
   - Rudders (yaw control)

2. **Secondary Flight Controls**
   - Wing sweep actuators
   - Flaps and slats
   - Speedbrakes

3. **Utility Systems**
   - Landing gear
   - Nose wheel steering
   - Arresting hook
   - Catapult nose tow

### Control Authority vs. Pressure

```
Pressure (PSI)    Control Authority
3000              100% (full rate, full deflection)
2000               95% (slightly reduced rate)
1800               90% (minimum for normal ops)
1100               50% (degraded - emergency threshold)
<800              <25% (inadequate for safe flight)
```

The model implements pressure-dependent control effectiveness through the `systems/hydraulics/*-system-pressure` binary flags.

## Key Properties (Property Tree)

### Input Properties

```
systems/hydraulics/flight-system-pump-failed        (0 or 1)
systems/hydraulics/combined-system-pump-failed      (0 or 1)
systems/hydraulics/emerg-system-pump-failed         (0 or 1)
systems/hydraulics/hyd-transfer-pump-switch         (0 or 1)
propulsion/engine[0]/n2                             (0-100%)
propulsion/engine[1]/n2                             (0-100%)
systems/electrics/ac-right-main-bus                 (volts)
```

### Output Properties

```
systems/hydraulics/flight-system-psi                (PSI)
systems/hydraulics/combined-system-psi              (PSI)
systems/hydraulics/flight-system-pressure           (0 or 1 flag)
systems/hydraulics/combined-system-pressure         (0 or 1 flag)
systems/hydraulics/emerg-pump-low-speed-psi         (PSI)
```

### Internal Calculation Properties

```
systems/hydraulics/bidi-combined-to-flight-system-factor
systems/hydraulics/bidi-flight-to-combined-system-factor
systems/hydraulics/flight-system-bleed
systems/hydraulics/elevator-bleed
```

## Constants (Tunable Parameters)

Defined in property tree at initialization:

```
systems/hydraulics/normal-pressure            3000 PSI
systems/hydraulics/bidi-normal-pressure       2000 PSI (transfer cap)
systems/hydraulics/bidi-activation            1800 PSI (transfer engage)
systems/hydraulics/bidi-cutoff-min-pressure    800 PSI (minimum for transfer)
systems/hydraulics/emerg-pump-auto-activation 1100 PSI (emergency on)
systems/hydraulics/emerg-pump-auto-deactivation 1400 PSI (emergency off)
```

These can be adjusted for different failure scenarios or testing.

## Design Philosophy

### Redundancy

The bidirectional transfer system provides true redundancy:
- Single engine operation: Full hydraulic capability
- Single pump failure: Automatic compensation
- Dual engine failure (windmill): Emergency backup

### Simplification vs. Reality

Model simplifications:

1. **No Accumulator Modeling**: Real F-14 has accumulators for pressure spikes
2. **Simplified Load Model**: Only elevator bleed modeled in detail
3. **No Thermal Effects**: Fluid temperature not simulated
4. **No Fluid Level**: Assumes adequate fluid in all reservoirs

These simplifications maintain flight control fidelity while reducing computational load.

### Electrical Interdependency

Emergency pump requires:
- AC electrical power (ac-right-main-bus)
- If electrical system fails, no emergency hydraulics
- Emphasizes importance of electrical system management

## Failure Modes and Effects

### Single Pump Failure

| Failed Pump | Effect | Mitigation |
|-------------|--------|------------|
| Flight | Pressure drop | Transfer from Combined |
| Combined | Pressure drop | Transfer from Flight |
| Emergency | Loss of backup | Still flyable on main pumps |

**Severity**: Minor (with transfer pump ON)

### Multiple Pump Failures

| Failure Combination | Effect | Flyability |
|---------------------|--------|------------|
| Flight + Combined | Total loss | Emergency pump required |
| Flight + Emergency | Degraded | Transfer from Combined OK |
| Combined + Emergency | Degraded | Transfer from Flight OK |
| All Three | Catastrophic | Not flyable |

### Transfer Pump Failure

If transfer pump switch fails OFF or pump fails:
- Single engine operation becomes critical
- No cross-system support
- Must rely on emergency pump if primary fails

### Electrical System Failure

Loss of AC power:
- Emergency pump inoperative
- Dual hydraulic failure becomes catastrophic
- Emphasizes importance of APU for emergency power

## Operating Procedures

### Normal Checklist

1. Verify hydraulic pressure indicators: Both systems 2800-3000 PSI
2. Check transfer pump switch: Normally ON
3. Monitor for pressure fluctuations during flight control checks
4. Caution lights: Should be dark

### Emergency Procedures

#### Single Hydraulic System Failure

1. Verify remaining system pressure adequate (>2000 PSI)
2. Confirm transfer pump ON
3. Monitor pressure in failed system (should rise via transfer)
4. Land as soon as practical
5. Avoid high-G maneuvers (increased hydraulic load)

#### Dual Hydraulic System Failure

1. **IMMEDIATE**: Check emergency pump activation
2. Verify AC power available
3. If emergency pump failed: Attempt restart
4. Minimize control inputs (conserve pressure)
5. Declare emergency
6. Land immediately at nearest suitable field

#### Flight Control Malfunction (Suspected Hydraulic)

1. Check hydraulic pressure indicators
2. If pressure low: Refer to hydraulic failure procedure
3. If pressure OK: Likely FCS or actuator fault
4. Use backup control modes if available
5. Land as soon as practical

## Maintenance Considerations

### Pressure Checks

Normal ground checks:
```
Engines OFF:  No hydraulic pressure
APU ON:       Emergency pump only (if tested)
Engine Start: Pressure rise with N2
Engine Idle:  3000 PSI both systems
```

### Leak Detection

Monitor for:
- Gradual pressure decay at idle
- Fluid level drop in reservoirs
- Visible leaks at actuators, lines, pumps
- Contamination in fluid samples

### Transfer Pump Testing

Ground test procedure:
1. Start one engine
2. Verify single system pressure
3. Activate transfer pump
4. Observe pressure rise in unpowered system
5. Should reach ~2500 PSI (83% of source)

## System Interactions

### With Flight Control System (FCS)

File: `f-14-fcs.xml`

FCS reads hydraulic pressure flags:
```
if systems/hydraulics/flight-system-pressure == 0:
  Limit control authority
  Reduce actuator rates
  Disable certain SAS modes
```

### With Engines

Hydraulic pumps directly coupled to engine N2:
- Immediate response to throttle changes
- Windmilling engines provide some hydraulic power
- Engine failure immediately affects hydraulic system

### With Electrical System

Emergency pump dependency:
```
Required: systems/electrics/ac-right-main-bus > 5V
Without:  Emergency pump inoperative
```

### With Warning System

File: `f-14-caution.xml`

Hydraulic pressure loss triggers:
- Master CAUTION light
- HYD PRESS caution light
- Aural warning tone

## Comparison with Real Aircraft

### Accurate Representations

- Dual system architecture
- N2-driven pumps with windmill capability
- Bidirectional transfer concept
- Emergency electric pump
- Pressure thresholds and logic

### Simplifications

- Accumulator dynamics not modeled
- Detailed utility system loads not modeled
- Fluid temperature effects ignored
- Line losses simplified to 83% factor
- No individual actuator modeling

### Purpose

This level of detail provides:
- Realistic emergency procedures training
- Accurate system failure consequences
- Proper dependency modeling for flight controls
- Sufficient fidelity for flight simulation

---

## References

1. **NAVAIR 01-F14AAD-1**: F-14A Aircraft Descriptive Data, Section 2.8, Pages 2-68 to 2-76
2. **NASA-TM-81833**: Flight control system schematics (indirect hydraulic information)
3. **F-14 Tomcat Model**: Implementation by Richard Harrison, http://zaretto.com/f-14

## Related Documentation

- **FCS.md**: Flight Control System (hydraulic consumers)
- **ENGINES.md**: Engine-driven hydraulic pumps
- **ELECTRICAL.md**: Emergency pump power requirements
- **LANDING_GEAR.md**: Hydraulic-powered gear system

---

*This hydraulic system simulation balances fidelity with computational efficiency, providing realistic behavior for flight training and emergency procedures while maintaining real-time performance in FlightGear.*
