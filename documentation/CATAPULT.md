# F-14 Tomcat Catapult System Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27

## Overview

The F-14 Tomcat is designed for carrier-based operations, requiring catapult-assisted takeoffs from aircraft carrier flight decks. The catapult system model simulates the launch bar attachment, holdback mechanism, and the powerful acceleration forces experienced during a catapult launch.

## Physical System Description

### Catapult Launch Bar

The nose gear includes a launch bar that extends forward to engage the catapult shuttle:

```
Launch Bar:
  Location: Nose gear strut
  Position: Forward-extending retractable bar
  Engagement: Hooks into carrier catapult shuttle
  Load: Entire aircraft weight + acceleration forces
```

### Holdback Mechanism

A tension bar holds the aircraft stationary while the catapult builds steam pressure:

```
Holdback:
  Material: Frangible link (designed to break)
  Purpose: Hold aircraft while engines spool to full thrust
  Release: Breaks when catapult fires (preset tension)
  Location: Rear attachment point on fuselage
```

### Nose Wheel Kneeling

The F-14 has a unique nose gear kneeling feature:

```
Kneel Function:
  Purpose: Lower nose to engage catapult shuttle
  Extension: Nose strut extends ~2 feet
  Activation: Pilot-commanded or automatic
  Ground clearance: Reduced significantly
```

Documented in: `LANDING_GEAR.md`

## Launch Sequence (Real Aircraft)

```
1. Taxi into position on catapult
   - Line up with shuttle track
   - Yellow shirt (director) guides

2. Extend launch bar
   - Cockpit control: Launch bar lever
   - Bar extends forward from nose gear
   - Visual confirmation (flight deck crew)

3. Engage catapult shuttle
   - Aircraft taxis slowly forward
   - Launch bar drops into shuttle slot
   - Visual check by deck crew

4. Extend nose strut (kneel)
   - Additional hook-up for safety
   - Increases launch bar angle
   - Optimizes catapult force vector

5. Tension holdback
   - Holdback bar attached to fuselage
   - Holds aircraft against catapult forces
   - Pre-tensioned to ~30,000 lbs

6. Flight control check
   - Full stick/rudder deflections
   - Visual check by flight deck crew
   - Wipe-out signal if problem detected

7. Tension catapult (steam build)
   - Catapult officer signals
   - Steam pressure builds to launch value
   - Aircraft strains against holdback

8. Throttles to full power
   - Both engines: Full afterburner
   - Hold brakes
   - Final engine checks

9. Salute (ready for launch)
   - Pilot salutes catapult officer
   - Final go/no-go decision
   - Night: Lights on signal

10. Catapult fires
    - Officer pushes launch button
    - Holdback breaks immediately
    - Steam drives shuttle and aircraft
    - Acceleration: 0 to 150+ knots in ~2 seconds
    - Distance: ~300 feet

11. End of stroke
    - Shuttle stops (water brake)
    - Aircraft continues (flying)
    - Gear up, positive rate of climb

12. Cleanup
    - Retract gear
    - Retract flaps
    - Climb away from ship
```

## Model Implementation

Located in: `Systems/catapult.xml`

### System Architecture

The catapult model uses a state machine with kinematic timing:

```xml
<system name="catapult">
  <channel name="Catapult">
```

Three main components:
1. Command switch (engage/disengage)
2. Timer (kinematic position over time)
3. Force calculation and application

### Launch Command (Cat Command Switch)

```xml
<switch name="systems/catapult/cat-cmd-norm">
```

Logic:

```
Default: 0 (inactive)

Resets to 0 when:
  cat-pos-norm > 0.999 (launch complete)

Sets to 1 when:
  cat-launch-cmd == 1 (pilot initiates)

Output: systems/catapult/cat-launch-cmd (latching)
```

This creates a latching behavior:
- Pilot sets launch command to 1
- System latches ON
- Remains ON until launch completes
- Automatically resets

### Catapult Timer (Position over Time)

```xml
<kinematic name="systems/catapult/cat-timer">
```

Kinematic profile:

```
Position    Time
  0.0       0.0 seconds (start)
  1.0       2.7 seconds (end)

Output: systems/catapult/cat-pos-norm (0 to 1)
```

This represents the catapult stroke:
- 0.0 = Start of stroke (shuttle at aft position)
- 1.0 = End of stroke (shuttle at forward position)
- Duration = 2.7 seconds (typical C-13 steam catapult)

Linear motion: Position advances linearly with time when commanded.

### Force Calculation

```xml
<pure_gain name="systems/catapult/cat-force">
  Input: inertia/weight-lbs
  Gain: 3.0
```

Catapult force calculation:

```
Cat Force = Aircraft Weight × 3.0

Example (70,000 lb aircraft):
  Force = 70,000 × 3 = 210,000 lbs

Resulting acceleration:
  a = F / m = 210,000 / (70,000 / 32.2) = 96.6 ft/sec²
  a ≈ 3.0 G
```

The factor of 3.0 provides approximately 3G acceleration, typical of carrier catapults.

### Force Application

```xml
<switch name="systems/catapult/cat-final">
```

Logic:

```
Default: 0 lbs (no force)

Force applied when ALL conditions met:
  1. cat-launch-cmd == 1 (launch commanded)
  2. cat-pos-norm < 0.999 (stroke not complete)
  3. cat-pos-norm > 0.0 (stroke in progress)
  4. Nose gear WOW ≠ 0 (nose gear on deck)

Applied force: systems/catapult/cat-force (210,000 lbs example)

Output: external_reactions/catapult/magnitude
```

The force is applied as an external reaction in JSBSim, pushing the aircraft forward.

### External Reaction Point

The catapult force must be defined in the main FDM file:

```xml
<external_reactions>
  <force name="catapult" frame="BODY">
    <location unit="IN">
      <x> 197 </x>  <!-- Nose gear location -->
      <y> 0 </y>
      <z> -94 </z>
    </location>
    <direction>
      <x> 1.0 </x>  <!-- Forward -->
      <y> 0.0 </y>
      <z> 0.0 </z>
    </direction>
  </force>
</external_reactions>
```

Force applied at nose gear, directed forward along body X-axis.

## Catapult Performance

### Launch Acceleration Profile

```
Time (sec)    Position    Velocity    Acceleration
  0.0         0.0         0 kts       3.0 G (initial)
  0.5         0.19        28 kts      3.0 G
  1.0         0.37        56 kts      3.0 G
  1.5         0.56        84 kts      3.0 G
  2.0         0.74        112 kts     3.0 G
  2.5         0.93        140 kts     3.0 G
  2.7         1.0         151 kts     3.0 G
```

Linear acceleration throughout stroke (constant force).

### End Speed Calculation

```
Catapult Stroke Length: ~300 feet (estimated)
Duration: 2.7 seconds
Average Velocity: 300 ft / 2.7 sec = 111 ft/sec = 65 knots

Final velocity (linear acceleration):
  V_final = 2 × V_avg = 130 knots (approximate)

Actual end speed depends on:
  - Aircraft weight (heavier = slower for same force)
  - Wind over deck (adds to effective airspeed)
  - Catapult setting (adjustable steam pressure)
```

Real carriers adjust catapult settings based on aircraft weight and wind.

### Weight Classes

Typical catapult settings for F-14:

```
Light (50,000 lbs):    Lower catapult setting, ~140 knots end speed
Medium (60,000 lbs):   Medium setting, ~135 knots end speed
Heavy (70,000 lbs):    High setting, ~130 knots end speed
Max Trap (74,000 lbs): Maximum setting, ~125 knots end speed
```

End speed must exceed stall speed with margin for safe flight.

## Operating Procedures

### Pre-Launch Checklist

```
1. Configuration
   - Landing gear: DOWN
   - Flaps: FULL (35°)
   - Slats: Extended
   - Wing sweep: 22° (minimum)
   - Speedbrake: RETRACTED

2. Weight and Balance
   - Check: Total weight < max catapult weight
   - Verify: CG within limits
   - Fuel: As required for mission (affects weight)

3. Systems
   - Hydraulics: Both systems pressure OK
   - Electrical: All buses powered
   - Flight controls: Check full deflection
   - Trim: Nose-up (takeoff trim)

4. Engines
   - Both engines: Started and stable
   - Instruments: Normal indications
   - Afterburners: Check operation (blip throttles)

5. Launch Bar
   - Extend launch bar (cockpit control)
   - Verify: Down and locked indication
   - Visual: Flight deck crew confirmation
```

### Launch Execution (Model)

In FlightGear:

```
1. Position aircraft on catapult
   - Align with deck centerline
   - Parking brake: SET

2. Set property:
   systems/catapult/cat-launch-cmd = 1
   (Or use keyboard shortcut/control)

3. Throttles: FULL AFTERBURNER
   - Both engines 100%
   - Hold brakes (parking brake)

4. Release brakes when catapult fires
   - Catapult begins accelerating aircraft
   - Maintain full throttle

5. Monitor airspeed
   - Airspeed increases rapidly
   - At rotation speed (~125 KIAS): Pitch gently to 12-15°
   - Positive rate: Gear UP

6. Climb away
   - Maintain 15° pitch until 500 ft AGL
   - Clean up (flaps, slats)
   - Reduce power after safe altitude
```

### Abort Scenarios

```
Abort before catapult fires:
  1. Throttles: IDLE
  2. Cat launch command: 0 (cancel)
  3. Signal deck crew: Wipeout (wave arms)
  4. Parking brake: SET
  5. Shutdown if required

Abort after catapult fires:
  - TOO LATE to abort
  - Must fly off deck
  - Eject if unable to fly
```

Once the catapult fires, the pilot is committed to launch.

## System Interactions

### With Holdback System

File: `Systems/holdback.xml`

Holdback must be modeled separately (if implemented):
- Holds aircraft stationary until catapult fires
- Breaks at predetermined tension
- Prevents premature aircraft movement

### With Landing Gear

File: `Systems/f-14b-landing-gear.xml`

Catapult requires:
- Nose gear DOWN and locked
- Nose gear WOW (Weight On Wheels) signal
- Kneeling function for proper engagement

If nose gear not down, catapult force not applied (safety interlock).

### With Flight Control System

File: `Systems/f-14-fcs.xml`

During launch:
- Pilot must maintain proper stick position (slight aft)
- Flight controls must be free (no jam detected)
- SAS should be engaged (stability augmentation)

### With Engines

File: `Systems/f-14b-engines.xml` or `f-14b-engines-TF-30.xml`

Launch requires:
- Both engines at maximum thrust (afterburner)
- Engine parameters normal (temp, RPM, pressure)
- Combined thrust ~50,000 lbs (adds to catapult force)

Total acceleration force:
```
F_total = Catapult + Engine Thrust
F_total = 210,000 + 50,000 = 260,000 lbs
a_total = 260,000 / (70,000/32.2) = 119.6 ft/sec² ≈ 3.7 G
```

## Key Properties (Property Tree)

### Control Properties

```
systems/catapult/cat-launch-cmd         (0 or 1) Pilot initiates launch
```

### Output Properties

```
systems/catapult/cat-cmd-norm           (0 or 1) Latched command
systems/catapult/cat-pos-norm           (0-1) Catapult stroke position
systems/catapult/cat-force              (lbs) Calculated force
external_reactions/catapult/magnitude   (lbs) Applied force to aircraft
```

### External Inputs (Read by System)

```
inertia/weight-lbs                      Aircraft total weight
gear/unit[0]/WOW                        Nose gear weight-on-wheels
```

## Emergency Procedures

### Cold Cat (Insufficient Catapult Power)

```
Symptoms:
  - Slow acceleration off catapult
  - Insufficient airspeed at deck edge
  - Aircraft sinking after leaving deck

Actions:
  1. Throttles: FULL AFTERBURNER (verify)
  2. Pitch: Hold shallow climb attitude
  3. Accelerate in ground effect
  4. Gear: UP immediately (reduce drag)
  5. Flaps: Retract gradually after safe speed
  6. If altitude insufficient: EJECT

Cause: Catapult malfunction or incorrect weight setting
```

### Hot Cat (Excessive Catapult Power)

```
Symptoms:
  - Very rapid acceleration (>4G)
  - Possible structural damage
  - Excessive end speed

Actions:
  1. Throttles: Maintain position
  2. Pitch: Gentle rotation (avoid over-G)
  3. Climb normally
  4. After safe altitude: Inspect aircraft
  5. Check flight controls for damage
  6. Land ASAP if anomalies

Cause: Catapult set for lighter aircraft
```

### Nose Gear Failure During Launch

```
Symptoms:
  - Loss of nose gear during catapult stroke
  - Aircraft pitches up violently
  - Possible loss of control

Actions:
  1. Throttles: FULL AFTERBURNER
  2. Pitch: Counter excessive pitch-up
  3. Gear: Attempt to retract (if possible)
  4. Accelerate and climb
  5. Emergency landing at divert field
  6. If uncontrollable: EJECT
```

### Engine Failure During Launch

```
Single engine failure:
  1. Throttle (good engine): FULL AFTERBURNER
  2. Rudder: Compensate for asymmetric thrust
  3. Pitch: Climb as able
  4. Gear: UP (reduce drag critical)
  5. Accelerate to safe speed
  6. Emergency pattern to land

Dual engine failure:
  - Extremely rare
  - Insufficient thrust to fly
  - EJECT immediately
```

## Limitations and Simplifications

### Model Limitations

1. **Constant Force**: Real catapults have non-linear acceleration (steam builds)
2. **No Weight Adjustment**: Real catapults adjust setting based on aircraft weight
3. **No Random Variation**: Every launch identical (no cold/hot cat variability)
4. **No Structural Limits**: Model doesn't break nose gear from over-stress
5. **Perfect Engagement**: No mis-engagement or launch bar failures
6. **No Holdback**: Holdback system not fully modeled (separate file)

### Validated Behavior

Despite simplifications, the model accurately represents:
- Launch acceleration (~3G sustained)
- Duration (~2.7 seconds)
- End speed (130-150 knots depending on weight and wind)
- Force application point (nose gear)
- Safety interlocks (gear down required)

## Comparison with Real Aircraft

### Accurate Representations

- Launch sequence timing
- Acceleration forces (3G)
- Nose gear attachment point
- Weight-proportional force
- Safety interlocks (gear down)

### Differences from Reality

```
Real Catapult:
  - Steam-driven shuttle (non-linear power)
  - Adjustable setting per aircraft weight
  - Holdback restraint (modeled separately)
  - Launch bar engagement critical
  - Deck crew coordination essential
  - Wave-off possible (deck foul, malfunction)

Model Catapult:
  - Idealized constant force
  - Fixed force multiplier (3.0)
  - Simplified holdback
  - Automatic engagement (property toggle)
  - No crew interaction required
  - No wave-off (launch always proceeds)
```

### Purpose

The model provides:
- Realistic launch experience (acceleration, duration)
- Proper procedures (configuration, throttle management)
- Emergency scenario training (engine failure on launch)
- Sufficient fidelity for flight simulation

Not intended for:
- Engineering analysis of catapult systems
- Detailed structural load calculations
- Flight deck crew training (deck handling)

## Related Systems

### Launch Bar and Kneeling

Documented in: `LANDING_GEAR.md`

The nose gear kneeling function is critical for catapult engagement. See that document for details on:
- Kneeling mechanism
- Strut extension/compression
- Launch bar deployment

### Holdback System

File: `Systems/holdback.xml`

The holdback system complements the catapult:
- Restrains aircraft during engine run-up
- Breaks when catapult fires
- Provides positive retention before launch

### Arresting Gear (Hook)

File: `Systems/hook.xml`

For landing (opposite of launching):
- Tailhook extends to catch arresting wire
- Rapid deceleration (opposite of catapult acceleration)
- Critical for carrier recovery

## Testing and Validation

### Ground Test

Not applicable (catapult only functions in flight simulation environment)

### Flight Test

```
Test Procedure:
  1. Position F-14 on carrier deck (catapult)
  2. Configuration: Landing configuration
  3. Engines: Full afterburner
  4. Initiate catapult launch (property or control)
  5. Monitor:
     - Acceleration: Should feel ~3G
     - Duration: Should be ~2.7 seconds
     - End speed: Should reach 130-150 KIAS
     - Aircraft should fly off deck safely

Expected Results:
  - Smooth acceleration
  - Sufficient end speed for flight
  - Proper rotation and climb
```

## References

1. **F-14 Flight Manual (NATOPS)**: Catapult launch procedures
2. **Carrier Aircraft Operations**: Standard Navy procedures
3. **JSBSim Documentation**: External reactions and force application
4. **Implementation**: Richard Harrison, catapult.xml

## Related Documentation

- **LANDING_GEAR.md**: Nose gear kneeling system
- **ENGINES.md**: Engine thrust during launch
- **FCS.md**: Flight control during takeoff

---

*The catapult system simulation provides a realistic carrier launch experience, capturing the essential dynamics of steam catapult operations while maintaining computational efficiency suitable for real-time flight simulation.*
