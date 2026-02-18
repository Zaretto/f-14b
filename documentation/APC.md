# F-14 Tomcat Approach Power Compensator (APC) Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Date**: December 15, 2020
**Last Updated**: 2026-01-27
**Reference**: NASA TM-X-81833, Figure 38 - Schematic of Autothrottle Control Law

## Overview

The Approach Power Compensator (APC) is an automatic throttle control system designed to maintain constant angle of attack during carrier approaches. By automatically adjusting engine thrust in response to pitch control inputs, glide slope corrections, and direct lift control (DLC) usage, the APC significantly reduces pilot workload during the demanding task of carrier landings.

## System Purpose

Carrier approaches in the F-14 require simultaneously managing:
- Angle of attack (airspeed)
- Glide slope (altitude)
- Line-up (lateral position)
- Aircraft attitude

The APC implements the "ball, angle of attack, lineup" (BAL) approach philosophy:
- **Pilot controls**: Glide slope with stick (pitch), lineup with rudder/stick (lateral)
- **APC controls**: Angle of attack (airspeed) with throttles automatically
- **Result**: Decoupled, easier approach task

## Physical System Description

### System Components

```
Inputs:
- Angle of Attack (AOA) probe
- Pitch rate gyro (q)
- Normal accelerometer (Nz)
- Stick position (elevator)
- DLC thumbwheel position
- Reference AOA setting (pilot selected)

Processing:
- Analog computer circuitry
- Control law computation
- Rate limiters and filters

Outputs:
- Throttle position command (PLA - Power Lever Angle)
- Engagement indicators
- Warning lights
```

### Control Modes

The APC can operate in multiple engagement states:

```
OFF:      APC inactive, manual throttle control
STANDBY:  APC monitoring, not commanding throttles
ACTIVE:   APC commanding throttles automatically
```

## Control Law Architecture

From NASA TM-X-81833 Figure 38, the APC implements a sophisticated multi-path control law:

![APC Block Diagram](https://i.imgur.com/gAbAGeE.png)

### Path V-1: AOA Error Integration

```
Input: α_u - α_ref

Where:
  α_u = α_indicated - 7.62 × q
  (Pitch rate compensated AOA)

Processing:
  1. AOA error = α_u - α_ref
  2. Gain = 3.5
  3. Dead zone: ±10 degrees (prevents actuation for large errors)
  4. Limiter: ±1 degree/sec
  5. Integration: 1/s (accumulator)

Output: V-1 (base throttle correction)
```

#### Pitch Rate Compensation

The 7.62 × q term compensates for pitch rate:
- During pitch-up: Reduces effective AOA (anticipates deceleration)
- During pitch-down: Increases effective AOA (anticipates acceleration)
- Units: 7.62 ft (effective moment arm from CG to AOA probe)

This makes the system predictive rather than purely reactive.

#### Dead Zone Logic

```
if |AOA error| > 10°:
  V-1 contribution = 0 (large error, APC cannot handle)
else:
  V-1 accumulates throttle correction
```

Large AOA errors indicate:
- System not engaged properly
- Major disturbance
- Pilot override needed

#### Integration and Trigger

```
Integrator resets when:
  systems/apc/trigger = -1
  (Occurs when APC switches from ACTIVE to OFF)

Otherwise:
  V-1 accumulates continuously
```

Located in: `f-14-apc.xml`, lines 53-106

### Path V-2: Acceleration Feedback

```
Input: (AOA error) - (50 × Nz)

Processing:
  1. Compute: error = (α_u - α_ref) - 50 × Nz
  2. Lead-lag filter: 1/(s+1)
  3. Scaled by active flag

Output: V-2 (acceleration compensation)
```

#### Rationale

Normal acceleration (Nz) indicates vertical acceleration:
- Positive Nz (down): Aircraft sinking, need more thrust
- Negative Nz (up): Aircraft climbing, need less thrust

The factor of 50 provides appropriate scaling between G-units and degrees.

#### Lead-Lag Filter

```
Transfer function: 1/(s+1)

Effect:
- Low-pass filter with time constant = 1 second
- Smooths rapid accelerometer noise
- Passes sustained acceleration changes
```

Located in: `f-14-apc.xml`, lines 108-143

### Path V-3: Control Input Feedforward

```
Inputs:
  1. DLC (Direct Lift Control) position
  2. Elevator position

DLC Component:
  DLC contribution = DLC_pos × gain

  Where gain =
    0.378 if DLC > 0 (positive/up)
    0.208 if DLC < 0 (negative/down)

Elevator Component:
  Elevator contribution = elevator_pos × 4.0

Total V-3-1 = (DLC component + Elevator component) × active_flag
```

#### Two-Stage Filtering

```
Stage 1: Lead filter
  10s / (10s + 1)

Stage 2: Lag filter
  1 / (0.5s + 1)

Net effect: High-pass characteristic
```

This makes V-3 respond primarily to *changes* in control position, not steady-state deflections.

#### Sign Inversion

```
Final V-3 = -1 × V-3-filtered

Rationale:
- Pitch up (positive elevator): Anticipate deceleration, reduce thrust
- Pitch down (negative elevator): Anticipate acceleration, increase thrust
- DLC up: Creating lift with spoilers, reduce thrust to maintain speed
- DLC down: Reducing lift, increase thrust
```

Located in: `f-14-apc.xml`, lines 145-181

### Throttle Command Summation

```
Total throttle command (PLA degrees):
  PLA = V-1 + V-2 - V-3

Limited to: 20° to 60°

Where:
  20° = Flight Idle
  60° = Intermediate power (no afterburner in APC mode)
```

The three paths work together:
- **V-1**: Long-term AOA correction (integrator)
- **V-2**: Medium-term acceleration response
- **V-3**: Short-term anticipation of control inputs

### Normalized Throttle Output

```
Normalized PLA = (PLA_deg - 20) / 45

Clips to: 0.0 to 1.0

Maps to engine throttle position:
  0.0 = Flight idle
  1.0 = Military (intermediate) thrust
```

Located in: `f-14-apc.xml`, lines 184-216

## Operating Characteristics

### Reference AOA Setting

Pilot selects reference AOA via cockpit control:

```
Typical Settings:
  Optimum AOA:  15.0° (on-speed)
  Slow:         15.5° (slightly slow)
  Fast:         14.5° (slightly fast)

Property: systems/apc/alpha-u-reference
```

The APC will command throttle to maintain this AOA precisely.

### Engagement Sequence

```
1. Pilot positions APC switch to STANDBY
   - System monitors but doesn't command throttles

2. Pilot trims aircraft to desired AOA with manual throttles

3. Pilot moves APC switch to ACTIVE
   - V-1 integrator begins at current state
   - System starts commanding throttles
   - Throttles move to match V-1 + V-2 - V-3

4. Small corrections occur automatically
   - Pitch input → Anticipatory throttle change (V-3)
   - AOA drift → Integrated correction (V-1)
   - Acceleration → Damped response (V-2)
```

### Steady-State Behavior

In trimmed flight with APC active:

```
If on-speed (α_u = α_ref):
  - V-1: Stable (no AOA error, integration stopped)
  - V-2: Zero (no acceleration, filtered to zero)
  - V-3: Zero (no control movement, high-pass filtered out)

  Result: Throttles hold steady position
```

### Dynamic Response Examples

#### Pitch Correction (Glide Slope)

```
Pilot action: Stick back (pitch up for glide slope correction)

Immediate (V-3):
  - Elevator deflection detected
  - V-3 goes negative (reduce thrust)
  - Anticipates deceleration from pitch-up

Short-term (V-2):
  - Pitch-up causes negative Nz (upward acceleration)
  - V-2 increases (adds thrust)
  - Compensates for climb

Long-term (V-1):
  - AOA begins to increase
  - V-1 integrates negative (reduce thrust)
  - Restores original AOA

Net result: Glide slope corrected, AOA maintained
```

#### DLC Correction (Precise Glide Slope)

```
Pilot action: DLC thumbwheel up (spoilers up for descent)

Immediate (V-3):
  - DLC deflection detected
  - V-3 goes negative (reduce thrust)
  - Anticipates that spoilers create drag

Short-term (V-2):
  - Spoilers create downward acceleration
  - V-2 responds to Nz change

Long-term (V-1):
  - AOA tends to increase (deceleration from drag)
  - V-1 compensates

Net result: Precise altitude correction, AOA maintained
```

#### Disturbance Rejection (Wind Gust)

```
Disturbance: Headwind gust (sudden airspeed increase)

Immediate:
  - AOA decreases below reference
  - V-3: No control input, no immediate change

Short-term (V-2):
  - Wind shear may cause Nz disturbance
  - V-2 provides damping

Long-term (V-1):
  - AOA error integrates
  - V-1 reduces throttle
  - Airspeed bleeds back to correct value

Net result: AOA restored automatically
```

## Integration with Other Systems

### Autopilot/AFCS Interaction

```
File: f-14b-AFCS.xml

When autopilot pitch mode active:
  - Autopilot commands elevator
  - APC sees elevator movement via V-3
  - APC adjusts throttle automatically
  - Decoupled control (pitch vs. speed)
```

This allows altitude hold or glide slope tracking without pilot manually managing speed.

### Direct Lift Control (DLC)

```
File: f-14-fcs.xml

DLC thumbwheel commands spoiler deflection:
  +45° down (spoilers up, downward lift)
  -45° up (spoilers down, neutral)

APC V-3 path:
  Automatically adjusts thrust for DLC drag/lift changes
```

DLC allows precise glide slope control (inches per second) while APC maintains speed.

### Engine Response

```
File: f-14b-engines.xml or f-14b-engines-TF-30.xml

APC commands throttle position:
  - Engines respond with natural lag (~2-3 seconds)
  - Thrust changes affect airspeed
  - Airspeed changes affect AOA
  - Closed-loop system

Engine limits:
  - APC limited to 20° - 60° PLA (no afterburner)
  - Insufficient for aggressive maneuvering
  - Designed for stabilized approach only
```

## Key Properties (Property Tree)

### Control Properties

```
systems/apc/active                  (0 or 1) Engagement flag
systems/apc/alpha-u-reference       (degrees) Target AOA
```

### Internal Computations

```
systems/apc/alpha-u                 (degrees) Pitch-rate compensated AOA
systems/apc/v-1-1                   (degrees) Raw AOA error × 3.5
systems/apc/v-1-2                   (deg/sec) Rate-limited error
systems/apc/v-1                     (degrees) Integrated correction
systems/apc/v-2-1                   (scaled) Acceleration term
systems/apc/v-2-2                   (scaled) Error - acceleration
systems/apc/v-2                     (filtered) Acceleration compensation
systems/apc/v-3-dh                  (scaled) Elevator contribution
systems/apc/v-3-dlc                 (scaled) DLC contribution
systems/apc/v-3-1                   (scaled) Sum of feedforward terms
systems/apc/v-3                     (filtered) Final feedforward
```

### Throttle Outputs

```
systems/apc/throttle-pla-deg        (degrees) Commanded PLA (20-60°)
systems/apc/throttle-pla-norm       (0-1) Normalized throttle command
```

### External Inputs (Read by APC)

```
aero/alpha-indicated-deg            Angle of attack sensor
velocities/q-aero-rad_sec           Pitch rate
accelerations/Nz                    Normal acceleration (G, body axis)
fcs/elevator-pos-deg                Elevator deflection
fcs/dlc-thumbwheel-pos-deg          DLC thumbwheel position
fcs/dlc-active                      DLC system engagement flag
```

## Operating Procedures

### Carrier Approach with APC

```
1. Carrier Pattern Entry
   - Speed: 250 KIAS
   - Altitude: 600 ft AGL
   - Gear: DOWN
   - Flaps: FULL
   - Hook: DOWN
   - APC: STANDBY

2. Approach Turn
   - Reduce speed to on-speed AOA (indexer green)
   - Manually adjust throttles for AOA
   - Wings level on final: 3/4 mile

3. APC Engagement
   - Trim aircraft to on-speed AOA
   - APC: ACTIVE
   - Verify throttles respond to pitch inputs
   - Hands on throttles (monitor)

4. Final Approach
   - Cross reference: Meatball, lineup, AOA
   - Corrections: Small pitch inputs only
   - APC maintains speed automatically
   - DLC for fine glide slope adjustments

5. In Close (Last 15 seconds)
   - Burble correction (power increase)
   - APC responds to pitch inputs
   - DLC for lineup and last-second glide slope

6. Touchdown
   - Throttles: IDLE (manual, override APC)
   - APC: OFF after landing
```

### Normal Configuration

```
Recommended Settings:
  Landing Gear:   DOWN
  Flaps:          FULL (35°)
  Slats:          Extended
  Wing Sweep:     22° (minimum)
  Hook:           DOWN (carrier) or UP (field)
  DLC:            ACTIVE
  APC:            ACTIVE (approaches) or OFF (pattern)
```

### Field Landing with APC

APC is useful for field approaches too:

```
Advantages:
  - Maintains precise approach speed
  - Reduces throttle workload
  - Consistent results

Procedure:
  - Engage APC on final (2-3 miles)
  - Set reference AOA for field speed (typically 13-14°)
  - Small pitch corrections for glide slope
  - Disengage APC at 50 ft AGL (flare)
```

## Emergency Procedures

### APC Malfunction (Runaway Throttles)

```
Symptoms:
  - Throttles moving without pilot input
  - Airspeed deviating from desired
  - APC light flickering or erratic

Actions:
  1. APC switch: OFF (immediate)
  2. Throttles: Manual control
  3. Trim airspeed manually
  4. Continue approach without APC
  5. Land normally

Difficulty:
  Increased workload (must manually manage throttle + pitch)
```

### APC Failure to Engage

```
Symptoms:
  - APC switch ACTIVE but no throttle movement
  - System appears inactive

Checks:
  1. Hydraulic pressure: OK? (Required for DLC input)
  2. Electrical power: OK? (CADC must function)
  3. AOA indicator: Working? (Bad probe = bad APC)

If no obvious cause:
  - Continue approach manually (no APC)
  - Increased workload
```

### APC Oscillation

```
Symptoms:
  - Throttles hunting (rapid back-and-forth)
  - Airspeed oscillating
  - Possible system instability

Possible causes:
  - Turbulence (normal, usually acceptable)
  - Bad AOA probe (erratic signal)
  - System malfunction (gain too high)

Actions:
  1. If oscillation severe: APC OFF
  2. If mild: Acceptable, monitor
  3. Land as soon as practical
```

### Waveoff with APC Active

```
Action: Bolter (miss wires) or waveoff call

Procedure:
  1. Throttles: FULL MILITARY (manual override)
     - Do NOT wait for APC to respond
     - Immediate manual intervention
  2. APC: OFF (pilot action or auto-disengage)
  3. Pitch: Climb attitude (~10° nose up)
  4. Gear: UP (after positive climb)
  5. Clean up (flaps/DLC) per SOP
```

The APC is NOT designed for waveoff thrust changes - too slow. Manual throttle override essential.

## Advantages and Disadvantages

### Advantages

```
+ Significantly reduces pilot workload
+ Decouples pitch and throttle tasks
+ More consistent approach performance
+ Allows precise DLC usage without speed penalty
+ Reduces "chasing" airspeed
+ Frees pilot to focus on lineup and glide slope
+ Better results in turbulence (automatic compensation)
```

### Disadvantages

```
- Adds system complexity
- Potential failure mode
- Pilot must monitor for malfunctions
- Not suitable for all approach types
- Requires proper setup and engagement
- Engine response lag can cause transients
- Must be disengaged for waveoff (delay)
```

### Pilot Acceptance

Historical note: The APC was highly valued by fleet pilots and significantly improved carrier landing grades, especially for new pilots still developing "throttle technique."

## System Tuning

The implementation uses carefully chosen gains from NASA TM-X-81833:

```
AOA Error Gain (V-1):          3.5
Rate Limit (V-1-2):            ±1.0 deg/sec
Acceleration Gain:             -50.0
DLC Gain (positive):           0.378
DLC Gain (negative):           0.208
Elevator Gain:                 4.0
V-3 Lead Time Constant:        10.0 seconds
V-3 Lag Time Constant:         0.5 seconds
```

These values were validated through flight test and simulation.

### Why These Gains?

```
AOA Error Gain (3.5):
  - Provides adequate correction authority
  - Not so high as to cause overshoot
  - Matches engine response characteristics

Acceleration Gain (-50):
  - Converts G-units to degree-equivalent
  - Scaling factor from empirical testing

DLC Asymmetric Gains:
  - DLC up (0.378): More thrust compensation needed
  - DLC down (0.208): Less compensation needed
  - Reflects asymmetric drag characteristics

Elevator Gain (4.0):
  - Anticipates speed change from pitch
  - Factor relates elevator angle to ΔV effect
```

## Comparison with Real Aircraft

### Accurate Representations

- Three-path control law architecture (V-1, V-2, V-3)
- Integration, acceleration feedback, feedforward structure
- Pitch rate compensation of AOA
- DLC integration
- Throttle limiting to sub-afterburner range
- Basic engagement logic

### Simplifications

- No actuator dynamics (real system has hydraulic servo lag)
- Perfect sensors (no noise, drift, or failures)
- Simplified engagement logic (real system has more interlocks)
- No turbulence filtering (real system has additional smoothing)

### Fidelity Assessment

The model captures the essential control law behavior accurately enough for:
- Procedural training (engagement, monitoring, disengagement)
- Workload reduction demonstration
- System failure consequences
- Integration with DLC and AFCS

## Testing and Validation

### Ground Test

```
Procedure:
  1. Engine: Start and idle
  2. APC: STANDBY
  3. Verify APC light illuminated (standby mode)
  4. APC: ACTIVE
  5. Manually move throttles (override)
  6. Observe APC attempting to move throttles back
  7. APC: OFF
```

### Flight Test

```
Procedure:
  1. Altitude: 5,000 ft AGL (safe altitude)
  2. Configuration: Landing (gear, flaps, hook)
  3. Trim: On-speed AOA manually
  4. APC: ACTIVE
  5. Test: Small pitch inputs
  6. Observe: Throttles adjust automatically
  7. Verify: AOA returns to reference
  8. Test: DLC inputs
  9. Observe: Throttle compensation
```

Expected results:
- Small pitch-up: Throttles reduce momentarily, then increase to restore AOA
- DLC up: Throttles reduce to compensate for spoiler drag
- System should feel "locked" on AOA

## References

1. **NASA TM-X-81833** (May 1, 1980)
   - Title: "Simulator results of an F-14A airplane utilizing an aileron-rudder interconnect during carrier approaches and landings"
   - Authors: W. W. Kelly and P. W. Brown
   - Section: Figure 38 - Schematic of Autothrottle Control Law
   - URL: https://ntrs.nasa.gov/citations/19800020867

2. **NASA-TM-81972** (December 1, 1981)
   - Title: "Limited evaluation of an F-14A airplane utilizing an aileron-rudder interconnect control system in the landing configuration"
   - Authors: W. W. Kelley, E. K. Enevoldson
   - Flight test validation of APC system
   - URL: https://ntrs.nasa.gov/citations/19820005275

3. **Implementation**: Richard Harrison, f-14-apc.xml
   - Based on NASA TM-X-81833 control law schematic
   - URL: http://zaretto.com/f-14

## Related Documentation

- **FCS.md**: Flight Control System (DLC, elevator inputs to APC)
- **ENGINES.md**: Engine response to APC throttle commands
- **AFCS.md**: Autopilot integration with APC
- **CADC.md**: AOA measurement system

---

*The APC system represents a sophisticated approach to reducing pilot workload during carrier landings. By automatically managing the speed (AOA) axis, it allows pilots to focus on the critical lineup and glide slope tasks, significantly improving safety and consistency.*
