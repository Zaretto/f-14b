# F-14 Tomcat Central Air Data Computer (CADC) Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27
**References**:
- NAVAIR 01-F14AAD-1 (F-14A Aircraft Descriptive Data)
- NASA TT F-15,406 "Improvement of Maneuverability at Low Speed" (W. Staudacher, 1972)

## Overview

The Central Air Data Computer (CADC) is a critical avionics component that processes flight parameters and automatically controls maneuvering flaps and slats to optimize aircraft performance across the flight envelope. The system significantly improves handling characteristics, particularly at high angles of attack and during high-G maneuvering.

## System Purpose

The CADC performs multiple functions:

1. **Automatic Maneuvering Flaps/Slats Control**: Deploys high-lift devices based on alpha and Mach number
2. **Flight Envelope Protection**: Prevents dangerous configurations (e.g., flaps at high Mach)
3. **Airframe Stress Monitoring**: Tracks G-loading and structural damage
4. **Flutter Detection and Modeling**: Simulates wing flutter at high speeds
5. **Angle of Attack Indexer Control**: Provides visual approach guidance
6. **Damage Modeling**: Tracks cumulative airframe damage from over-G and overspeed

## Physical System Description

The CADC is an analog computer (later aircraft have digital CADC) that receives inputs from:

### Input Sensors

```
Primary Sensors:
- Pitot-static system (airspeed, altitude, Mach number)
- Angle of Attack (AOA) probe (indicated alpha)
- Accelerometers (normal acceleration, G-loading)
- Landing gear position switches
- Flap position sensors
- Wing sweep position sensors
```

### Output Commands

```
Actuator Commands:
- Main flap position (0° to 35°)
- Auxiliary flap position (0° to 35°)
- Slat position (0° to 17°)

Indicator Outputs:
- AOA indexer lights (slow/on-speed/fast)
- Master caution/warning triggers
```

## Maneuvering Flaps and Slats System

### Design Philosophy

Traditional high-lift devices (flaps/slats) are used only for takeoff and landing at low speeds. The F-14's maneuvering system extends this concept:

**Automatic deployment during combat maneuvering improves:**
- Turn rate (increased CL at given airspeed)
- Departure resistance (delayed stall)
- Control authority at high alpha
- Overall maneuverability in the "corner of the envelope"

### Operating Envelope

From NAVAIR 01-F14AAD-1 Figures 2-51 and 2-52 (Page 2-93):

#### Full Extension Limits

Maximum Mach number for full maneuvering flap/slat deployment:

```
Altitude (ft)    Max Mach
    0            0.511
  5,000          0.561
 10,000          0.621
 15,000          0.679
 20,000          0.749
 25,000          0.801
 30,000          0.852
 35,000          0.860
 40,000          0.864
```

Above these limits, flaps/slats will not deploy (or will retract if deployed).

#### Partial Extension Limits

Maximum Mach for partial extension (higher threshold):

```
Altitude (ft)    Max Mach
    0            0.581
  5,000          0.630
 10,000          0.698
 15,000          0.770
 20,000          0.830
 25,000          0.859
 30,000          0.865
 35,000          0.870
 40,000          0.877
```

Between partial and full extension Mach limits, reduced flap deployment occurs.

### Extension Schedule (Alpha-Based)

The system monitors angle of attack and automatically deploys flaps/slats:

#### Extension Trigger (systems/cadc/manuv-flapslat-extension-aoa)

```
Mach     AOA for Extension
0.00     10.45°
0.40     10.45° (hold)
0.45      8.12° (transition)
0.50      7.63°
0.60      6.08°
0.70      5.65°
0.80      5.48°
0.86      5.49°
```

When indicated AOA exceeds these values, system begins deployment.

#### Retraction Trigger (systems/cadc/manuv-flapslat-retraction-aoa)

```
Mach     AOA for Retraction
0.00     7.62°
0.38     7.62° (hold)
0.40     5.30° (transition)
0.50     4.81°
0.60     3.71°
0.70     2.74°
0.80     2.37°
0.87     2.73°
```

When AOA drops below these values, system retracts flaps/slats.

**Hysteresis**: The separation between extension and retraction thresholds prevents rapid cycling during maneuvering.

### Deployment Logic

Located in: `f-14b-cadc.xml`, lines 149-197

```
Maneuvering system ACTIVE when:
  1. Landing flaps ≤ 13° (not in landing config)
  2. AOA > extension threshold
  3. Mach ≤ fully-extended-max-mn
  4. Landing gear UP
  5. Wing sweep ≤ 50° (can't deploy at high sweep)

Maneuvering system INACTIVE when:
  - Any of above conditions violated
  - System holds current position during transitions
```

Once active, the system remains engaged until AOA drops below retraction threshold, providing stable operation during hard maneuvering.

### Flap Position Calculation

From NAVAIR 01-F14AAD-1 Section 2.21.2.4 (Page 2-92):

```
Maneuvering Flap Position:

AOA (deg)    Mach 0.50    Mach 0.60    Mach >0.90
    0         0° (0%)      0° (0%)      0° (0%)
    6         7% deflect   7% deflect   0°
   20        28.6%        18.6%        0°
   35        28.6%        18.6%        0°
   50+       28.6%        18.6%        0°

(Values are normalized: 1.0 = 35° full deployment)
```

Above Mach 0.90, maneuvering flaps locked at 0° regardless of AOA (aerodynamic limits).

### Slat Position Calculation

```
Maneuvering Slat Position:

AOA (deg)    Mach 0.45-0.80    Mach >0.90
    0         0° (0%)           0°
    8        7.6% (1.3°)       0°
   10       13.5% (2.3°)       0°
   20       42.9% (7.3°)       0°
   25+      42.9% (7.3°)       0°

(Values normalized: 1.0 = 17° full slat deployment)
```

Slats deploy more gradually than flaps, providing progressive improvement in high-alpha handling.

### Wing Sweep Interlock

```
Wing Sweep    Slat Availability
≤ 50°         Full range (0° to 17°)
> 50°         Locked at 0° (retracted)

Wing Sweep    Flap Availability
≤ 50°         Full auto schedule
> 50°         Manual only (if overridden)
```

This prevents flap/slat deployment with aft wing sweep where effectiveness is minimal and structural loads are excessive.

## Airframe Stress and G-Loading

### G-Load Monitoring

Located in: `f-14b-cadc.xml`, lines 314-365

The CADC continuously monitors structural loads:

```
Current G-Loading Calculation:
  Raw G = accelerations/Nz - 1.0
  (Nz is positive downward in body axis)

  Filtered through sensor with:
    - Time delay: 0.01 seconds
    - Lag filter: 4.5 coefficient
```

This filtering smooths transient accelerometer noise while tracking sustained G-loads.

### Wing Sweep Effect on Structural Strength

As wings sweep aft, they become more resistant to bending:

```
Wing Strength Factor = 1 / cos⁴(sweep - 22°)

Sweep (deg)    Strength Factor
    22         1.000 (baseline)
    25         0.995
    30         0.962
    40         0.818
    50         0.608
    60         0.386
    70         0.201
```

At maximum sweep (68°), the wing can withstand significantly higher G-loads before structural damage.

```
Effective G-Loading (for damage):
  Wing G = Current G × Strength Factor
```

Example: At 10G with 50° sweep:
```
Wing G = 10 × 0.608 = 6.08G equivalent stress
```

This allows higher-G maneuvering with swept wings.

### G-Limit Envelope

Design limits (configurable):

```
Maximum Positive G:  +6.5G (22° sweep, clean)
                     +8.5G (60° sweep)
Minimum Negative G:  -3.0G (22° sweep)
                     -4.5G (60° sweep)
```

These are "structural" limits. Pilot physiological limits are lower (~9G with G-suit).

### Damage Accumulation

The system tracks permanent wing bending damage:

#### Positive G Damage (Beyond +6.5G baseline)

```
When: Wing G-loading > max-g limit

Damage Rate = (Wing G - max-g) × permanent-bend-factor × dt

Accumulation:
  - Filtered through lag filter (c1 = 0.8)
  - Damage total ratchets (never decreases)
  - Capped at 1.0 (100% damage)
```

#### Negative G Damage (Beyond -3.0G baseline)

```
When: Wing G-loading < min-g limit

Damage Rate = (min-g - Wing G) × permanent-bend-factor × dt

Same accumulation as positive G damage
```

#### Wing Bend Effect

```
Total Wing Bend =
  + Permanent damage (cumulative)
  + Current G-loading × distortion-factor
  + Flutter contribution (high speed)

Output: fcs/wing-fold-pos-norm (visual wing bending in model)
```

Visible wing flexing in the 3D model reflects actual structural stress.

## Wing Asymmetry and Catastrophic Damage

Located in: `f-14b-cadc.xml`, lines 497-591

### Ultimate G-Limits

Beyond "structural" limits are "ultimate" limits:

```
Ultimate Maximum G:  ~9.0G (rough estimate)
Ultimate Minimum G:  ~-5.0G
```

Exceeding these with accumulated damage triggers catastrophic failure.

### Wing Failure Logic

```
Wing Tear-Off Occurs When:
  1. Accumulated bend damage > 0.1 (10%)
  AND
  2. Current wing G > ultimate-max-g
     OR Current wing G < ultimate-min-g

Result:
  - Random selection: Left or right wing fails
  - Stored in systems/flyt/wing-asymmetry-action
    (-1 = left wing, 0 = none, +1 = right wing)
```

### Aerodynamic Consequences

When a wing tears off:

```
Roll Moment:
  CLDasym = wing-asymmetry × ClSweepFactor × 0.076
  (Massive asymmetric rolling moment)

Drag Increase:
  CdDasym = wing-asymmetry² × CdBAS × -0.3
  (Squared to remove sign, major drag increase)

Lift Loss:
  Substantial lift loss on failed side
```

**Result**: Aircraft enters uncontrollable roll/spin. Recovery impossible.

### Damage Reset

```
Property: systems/flyt/damage-reset
  Set to 1: Resets all accumulated damage
  Automatically returns to 0

Used for:
  - Starting new flight
  - Scenario reset
  - Post-maintenance "repair"
```

## Flutter Modeling

Located in: `f-14b-cadc.xml`, lines 217-311

High-speed flutter is a dangerous aeroelastic phenomenon where aerodynamic forces excite natural structural frequencies.

### Flutter Speed

```
Property: systems/flyt/flutter-onset-ias
  Default: ~450 knots IAS (configurable)

Beyond this speed, flutter begins
```

### Flutter Characteristics

```
Pulsation Frequency:
  2 × π × 3.0 = 18.85 rad/sec (~3 Hz)

Amplitude Growth:
  Linear with overspeed up to full-flutter-ias
  Then constant maximum amplitude

Components:
  1. Pitch oscillation (small amplitude)
  2. Wing bending oscillation (large amplitude)
```

### Flutter Pitch

```
Flutter Pitch = 0.0008 × (IAS - onset_IAS) × sin(ω × time)

Small pitch oscillation, uncomfortable but not dangerous
```

### Flutter Wing Bending

```
Flutter Bend = amplitude × sin(ω × time)

Where amplitude grows from 0 at onset to max-bend-amplitude at full-flutter-ias

Effect:
  - Added to wing-fold-pos-norm (visible bending)
  - Stress cycles the structure
  - Prolonged flutter causes fatigue damage
```

### Flutter Time Integration

```
systems/flyt/flutter-time:
  Increments by dt only while IAS > onset-ias
  Used to drive sinusoidal oscillation
  Resets to 0 when speed drops below onset
```

This creates smooth flutter onset and cessation as speed varies.

## Flap and Slat Damage Modeling

Located in: `f-14b-cadc.xml`, lines 625-835

### Flap Speed Limits

```
Flap Position    Max Speed (knots IAS)
    0° - 10°     390 KIAS
   10° - 15°     390 KIAS
   15° - 35°     270 KIAS (landing flaps)
```

Real aircraft limit: 225 KIAS with flaps >4° (per NAVAIR 01-F14AAP-1 Figure 2-42).

### Maneuvering Flap Overspeed

```
Damage occurs when:
  - Mach > manuv-flap-fully-extended-max-mn
  - Maneuvering flaps deployed (> 0°)

Damage Rate:
  [(Mach - limit) / 1.0]² / damage-time × dt

Rapid damage accumulation with large overspeeds
```

### Damage Effects

```
Damage Factor = 1.0 - total_damage
  (0.0 = 100% damaged, 1.0 = undamaged)

Applied to:
  - fcs/flap-pos-deg-effective
  - fcs/aux-flap-pos-deg-effective

Result: Reduced aerodynamic effectiveness
```

### Flap Jamming

```
When damage > 50%:
  systems/flyt/flaps-jammed = 1
  Flap position frozen (cannot move)

When damage > 70%:
  systems/flyt/slats-jammed = 1
  Slat position frozen
```

Jammed flaps require emergency procedures and significantly limit landing speeds.

## AOA Indexer System

Located in: `f-14b-cadc.xml`, lines 858-931

The AOA indexer provides visual approach speed guidance to the pilot.

### Indexer Lights

Three lights visible on canopy bow and external indexer:

```
SLOW (Amber Chevron):
  Illuminates when: aero/alpha-indicated-deg ≥ slow-deg-start

ON-SPEED (Green Circle):
  Illuminates when:
    optimum-deg-start ≤ aero/alpha-indicated-deg ≤ optimum-deg-end

FAST (Red Chevron):
  Illuminates when: aero/alpha-indicated-deg ≤ fast-deg-end
```

### Indexer Activation Logic

```
Indexer operates when:
  1. AC essential bus powered (electrical requirement)
  2. Landing gear DOWN
  3. Tailhook deployed (or bypass field engaged)

Special cases:
  - Master test lights: All three illuminate
  - Electrical failure: All dark
```

### Approach AOA Targets

Typical values (configurable per variant):

```
SLOW threshold:    15.5° indicated
ON-SPEED range:    14.5° - 15.5° indicated
FAST threshold:    13.5° indicated
```

On-speed AOA provides optimal approach performance:
- Adequate margin above stall
- Good control response
- Proper sink rate for carrier landing

### Flashing Logic

```
Property: systems/electrics/aoa-indexer-light-off

When tailhook up but gear down:
  Indexer enters "field mode"
  Lights flash (state alternates)

When tailhook down and gear down:
  Indexer steady (carrier mode)
```

This reminds pilot that hook is not down for field landing.

## Key Properties (Property Tree)

### Inputs (Read by CADC)

```
aero/alpha-deg                          Aerodynamic angle of attack
aero/alpha-indicated-deg                Indicated AOA (with probe error)
velocities/mach                         Mach number
velocities/vc-kts                       Calibrated airspeed (knots)
position/h-sl-ft                        Altitude above sea level
gear/gear-cmd-norm                      Landing gear command (0 or 1)
fcs/flap-cmd-norm                       Manual flap command
fcs/wing-sweep-cmd                      Wing sweep command (0-1)
accelerations/Nz                        Normal acceleration (G)
```

### Outputs (Written by CADC)

```
systems/cadc/manuv-active                          (0 or 1)
systems/cadc/manuv-flap-extension-cmd              (0-1 normalized)
systems/cadc/manuv-slat-extension-cmd              (0-1 normalized)
systems/flyt/current-g-loading                     (G units)
systems/flyt/wing-bend-damage-total                (0-1)
systems/flyt/flaps-main-damage-total               (0-1)
systems/flyt/slats-damage-total                    (0-1)
systems/flyt/wing-asymmetry-action                 (-1, 0, +1)
systems/flyt/flutter-bend                          (rad or ft)
systems/flyt/wing-distortion                       (normalized)
fcs/flap-pos-deg-effective                         (degrees)
fcs/aux-flap-pos-deg-effective                     (degrees)
systems/electrics/aoa-indexer-slow                 (0 or 1)
systems/electrics/aoa-indexer-onspeed              (0 or 1)
systems/electrics/aoa-indexer-fast                 (0 or 1)
position/aircraft-on-ground                        (0 or 1)
```

## System Interactions

### With Flight Control System

The FCS reads CADC outputs:

```
File: f-14-fcs.xml

FCS uses:
  - manuv-flap-extension-cmd → actual flap deflection
  - manuv-slat-extension-cmd → actual slat deflection
  - wing-distortion → visual model position
  - damage flags → control authority limits
```

### With Aerodynamics

The aerodynamic model uses:

```
File: f-14a.xml

Aero model uses:
  - fcs/flap-pos-deg-effective (instead of raw position)
  - fcs/aux-flap-pos-deg-effective
  - systems/flyt/wing-asymmetry → asymmetric forces
  - fcs/wing-fold-pos-norm → visual wing bend
```

### With Electrical System

```
File: f-14b-electrical.xml

CADC requires:
  - AC essential bus for AOA indexer
  - System operational without electrical (analog computer)
  - Loss of electrical: Lose indexer, CADC continues
```

## Operational Procedures

### Normal Operations

1. **Takeoff**: Landing flaps set manually (typically 10°)
2. **Climb**: Retract flaps, CADC inactive (low AOA)
3. **Cruise**: CADC monitoring, maneuvering flaps retracted
4. **Combat Maneuvering**:
   - Hard turn: AOA increases
   - CADC automatically deploys maneuvering flaps/slats
   - Turn performance improves
   - System retracts when AOA decreases
5. **Approach**: Landing flaps extended manually, CADC inhibited
6. **Landing**: Follow AOA indexer for proper approach speed

### Emergency Procedures

#### Flap Overspeed

```
Indications:
  - Master CAUTION
  - REDUCE SPEED light (if >225 KIAS with flaps >4°)

Actions:
  1. Throttles: Idle
  2. Speedbrakes: Extend (if safe)
  3. Reduce speed below placard limit
  4. Check flap position indicators for damage
  5. If flaps damaged: Plan high-speed landing
```

#### Flap Asymmetry

```
Indications:
  - Flap asymmetry light (10-second if disagree, 3-second if lockout)
  - Rolling tendency
  - Abnormal pitch moment

Actions:
  1. Flap handle: EMER UP (override CADC)
  2. Wing sweep: Consider aft sweep for stability
  3. Land at higher speed
  4. Expect drift toward flap-down side
```

#### Over-G Event

```
After exceeding G limits:
  1. Note maximum G on standby G-meter
  2. Reduce maneuvering intensity
  3. Check for control anomalies (damage)
  4. Inspect wings visually (if possible)
  5. Land as soon as practical
  6. Ground inspection required before next flight

If control issues develop:
  1. Suspect structural damage
  2. Gentle control inputs
  3. Declare emergency
  4. Land immediately
```

#### Catastrophic Wing Failure

```
If wing tears off (asymmetry develops):
  - Aircraft UNRECOVERABLE
  - Eject immediately
  - Do not delay
```

This is modeled for realism but should be avoided through proper airmanship.

## Testing and Validation

### Ground Tests

```
CADC function test:
  1. Master test switch: ON
  2. AOA indexer: All three lights illuminate
  3. Master test switch: OFF
  4. AOA indexer: All lights dark (gear up)

Flap system test:
  1. Landing gear: DOWN
  2. Flap handle: Extended positions
  3. Observe flap position indicators
  4. Check for smooth, symmetric operation
```

### Flight Tests

```
Maneuvering flap deployment:
  1. Altitude: 10,000 ft
  2. Speed: 350 KIAS
  3. Wing sweep: 22°
  4. Establish ~6G turn
  5. Monitor AOA increase
  6. Observe maneuvering flaps deploy automatically
  7. Note improved turn rate
  8. Reduce G, observe retraction
```

## Limitations and Notes

### Model Simplifications

1. **No Pneumatic System**: Real CADC has pneumatic inputs, model uses direct air data
2. **Perfect Sensors**: No sensor lag or failure modes (except electrical)
3. **Idealized Deployment**: Real hydraulic actuator dynamics simplified
4. **No CADC Failure Modes**: Computer always operates (unless testing failure scenarios)

### Damage System Purpose

The damage modeling serves multiple purposes:
- **Training**: Teaches consequences of exceeding limits
- **Realism**: Proper flight envelope respect
- **Challenge**: Adds consequence to aggressive flying
- **Recovery Scenarios**: Practice emergency procedures

Damage can be disabled via property: `systems/flyt/damage-enabled = 0`

## References

1. **NAVAIR 01-F14AAD-1**: F-14A Aircraft Descriptive Data
   - Section 2.21: Maneuvering Flaps and Slats
   - Figures 2-51, 2-52: CADC operating envelope (Page 2-93)
   - Figure 2-48: Wing autosweep schedule (Page 2-87)

2. **NASA TT F-15,406** (1972): W. Staudacher, "Improvement of Maneuverability at Low Speed"
   - Theoretical basis for maneuvering high-lift systems

3. **NAVAIR 01-F14AAP-1**: F-14A Flight Manual
   - Figure 2-42: Reduce Speed Light logic
   - Section 2.21.1.1.1: Emergency Flap procedures

4. **NASA-TM-101717** (July 1990): "Flutter Clearance to the F-14A Variable-Sweep Transition Flight Experiment Airplane - Phase 2"
   - Flutter characteristics and testing

## Related Documentation

- **AERODYNAMICS.md**: How CADC outputs affect forces and moments
- **FCS.md**: Integration with flight control system
- **ELECTRICAL.md**: Power requirements for AOA indexer

---

*The CADC system represents sophisticated 1970s avionics technology, providing automated flight envelope optimization. This simulation captures the essential behavior while adding comprehensive structural integrity modeling for training and realism.*
