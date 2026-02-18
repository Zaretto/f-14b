# F-14 Tomcat Landing Gear System Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27

## Overview

The F-14 Tomcat features a robust tricycle landing gear configuration designed for the demanding environment of carrier operations. A unique feature is the nose gear "kneeling" capability, which allows the nose to be raised or lowered independently for catapult operations and deck storage.

## Physical System Description

### Landing Gear Configuration

```
Type: Tricycle (nose and main gear)

Nose Gear:
  Location: Forward fuselage (x = 197")
  Wheels: Single nose wheel
  Steering: Up to 89° (shimmy damper at high speed)
  Retraction: Aft into nose bay

Main Gear:
  Location: Fuselage (x = 451")
  Separation: 196" (8.17 feet) between wheels
  Wheels: Single wheel per strut
  Retraction: Aft into fuselage/wing glove
```

### Gear Specifications

From FDM file `f-14a.xml`:

```
Nose Gear:
  Position:  x=197", y=0", z=-94.01"
  Spring:    42,500 lbs/ft
  Damping:   2,000 lbs/ft/sec
  Friction:  μs=0.8, μd=0.5, μr=0.02
  Steering:  ±89° maximum
  Brake:     NOSE group (normally not used)

Left Main Gear:
  Position:  x=451", y=-98", z=-102.1"
  Spring:    42,500 lbs/ft
  Damping:   2,000 lbs/ft/sec
  Friction:  μs=0.8, μd=0.5, μr=0.02
  Steering:  0° (fixed)
  Brake:     LEFT group

Right Main Gear:
  Position:  x=451", y=98", z=-102.1"
  Spring:    42,500 lbs/ft
  Damping:   2,000 lbs/ft/sec
  Friction:  μs=0.8, μd=0.5, μr=0.02
  Steering:  0° (fixed)
  Brake:     RIGHT group
```

### Hydraulic Operation

Landing gear extension/retraction powered by hydraulic system:

```
Hydraulic Requirements:
  - System pressure: >1800 PSI for normal operation
  - Emergency: Gravity drop with blow-down bottles
  - Doors: Hydraulically actuated
  - Uplock/Downlock: Hydraulic mechanical locks

Actuation Time:
  Extension: ~7 seconds (hydraulic)
  Emergency: ~10 seconds (gravity + blow-down)
  Retraction: ~10 seconds (hydraulic)
```

Documented in: `HYDRAULICS.md`

## Nose Wheel Kneeling System

The F-14's unique nose gear kneeling system allows three distinct positions:

### Kneeling Positions

Located in: `Systems/f-14b-landing-gear.xml`, lines 12-63

```
Position 1: KNEEL (Extended)
  - Nose strut extends ~1.87 feet
  - Lowers nose significantly
  - Used for: Catapult attachment, maintenance

Position 2: NORMAL
  - Standard taxi/takeoff/landing position
  - Neutral strut position
  - Normal ground clearance

Position 3: HIGH (Compressed)
  - Nose strut compresses ~1.17 feet
  - Raises nose significantly
  - Used for: Deck storage, reduced profile
```

### Kinematic Implementation

```xml
<kinematic name="Nose strut kneeling">
  <input>fcs/gear-kneel-dmd</input>
  <traverse>
    <setting>
      <position>-1.167</position>  <!-- HIGH -->
      <time>5</time>                <!-- 5 seconds -->
    </setting>
    <setting>
      <position>0</position>        <!-- NORMAL -->
      <time>5</time>
    </setting>
    <setting>
      <position>1.867</position>    <!-- KNEEL -->
      <time>5</time>
    </setting>
  </traverse>
  <output>gear/unit[0]/base-compression-ft</output>
</kinematic>
```

The system provides smooth transition between three positions over 5 seconds each.

### Command Scaling

```xml
<aerosurface_scale name="Adjusted strut compression feet">
  Input: fcs/gear-kneel-cmd (0, 1, 2)

  Range:
    Min: -0.8 (compressed/high)
    Max:  0.833 (extended/kneel)

  Output: fcs/gear-kneel-dmd (normalized)
```

Pilot command (0, 1, 2) maps to physical strut position:
- 0 → HIGH position
- 1 → NORMAL position
- 2 → KNEEL position

### Compression Calculation

The total gear compression (for visual model) combines:

```
Total Compression =
  + gear/unit[0]/compression-ft (weight-based)
  + gear/unit[0]/base-compression-ft (kneel position)

Output: gear/unit[0]/compression-adjusted-ft
```

This allows the gear to:
- React to aircraft weight dynamically
- Add/subtract kneeling position offset
- Provide realistic strut appearance

### Visual Position

```xml
<fcs_function name="gear/unit[0]/z-position">
  Output = z-position-base + (base-compression-ft × 12)

  Converts feet to inches for 3D model positioning
```

The nose gear 3D model position updates in real-time based on kneeling position.

## Kneeling System Operations

### Normal Taxi/Takeoff

```
Configuration:
  Kneel Command: NORMAL (1)
  Strut Position: 0 feet offset
  Ground Clearance: Standard

Appearance:
  Nose at normal height
  Adequate prop clearance
  Stable taxi characteristics
```

### Catapult Operations

```
Pre-Launch:
  1. Taxi onto catapult
  2. Align with shuttle
  3. Kneel Command: KNEEL (2)
  4. Wait 5 seconds for extension
  5. Nose lowers ~1.87 feet
  6. Launch bar engages shuttle
  7. Holdback attached

Purpose:
  - Optimizes launch bar angle
  - Ensures positive shuttle engagement
  - Reduces stress on nose gear
  - Standard for all catapult launches

Post-Launch:
  - Kneel Command: NORMAL (automatic or manual)
  - Strut returns to normal during climb
```

Documented in: `CATAPULT.md`

### Deck Storage (High Position)

```
Configuration:
  Kneel Command: HIGH (0)
  Strut Position: -1.17 feet offset (compressed)
  Ground Clearance: Reduced

Appearance:
  Nose raised significantly
  Tail closer to deck
  Reduced vertical profile

Purpose:
  - Lower profile for hangar deck
  - Reduces space required
  - Standard for long-term deck parking
  - Aircraft "squats" on deck
```

### Maintenance Access

```
Configuration:
  Kneel Command: KNEEL (2) or HIGH (0)

Kneel Position (nose low):
  - Access to nose radar
  - Access to forward avionics
  - Access to cockpit (easier boarding)

High Position (nose high):
  - Access to engines
  - Access to tail systems
  - Better access to rear fuselage
```

## Landing Gear Limits and Restrictions

### Speed Limits

From F-14 NATOPS:

```
Maximum Gear Extension Speed:  280 KIAS
Maximum Gear Down Speed:       280 KIAS
Maximum Braking Speed:
  - Heavy (51,000 lbs):       145 KIAS
  - Light (46,000 lbs):       165 KIAS
Maximum Tire Speed:            190 KIAS
```

Exceeding these speeds can cause:
- Structural damage to gear
- Blown tires
- Hydraulic leaks
- Door damage

### Weight Limits

```
Maximum Landing Weight:  Maximum Trap Weight (MTV)
  F-14A/B: 54,000 lbs (normal)
           58,000 lbs (emergency)

Sink Rate Limits:
  Normal: 10 ft/sec (600 ft/min)
  Maximum: 18 ft/sec (1,080 ft/min) - carrier landing

Exceeding limits can cause:
  - Gear collapse
  - Tire failure
  - Structural damage
```

### Crosswind Limits

```
Maximum Crosswind Component:
  Takeoff: 20 knots (steady)
           30 knots (gust)
  Landing: 20 knots (steady)
           30 knots (gust)

Factors:
  - Nosewheel steering authority
  - Rudder effectiveness at low speed
  - Wing area (side load)
```

## Ground Handling

### Nosewheel Steering

The F-14 has exceptional nosewheel steering authority:

```
Steering Range: ±89° (nearly perpendicular)

Control:
  - Rudder pedals: Limited authority (~10°)
  - NWS button: Full authority (±89°)
  - Engaged via button on stick

Shimmy Damper:
  - Automatically reduces steering at speed
  - Limits oscillation at high taxi speeds
  - Full authority below 40 knots
  - Reduced above 60 knots
```

### Taxi Characteristics

```
Turning Radius (sharp turn):
  - With full nosewheel steering: Very tight (~50 ft radius)
  - Without NWS (rudder only): Wide (~200 ft radius)

Differential Braking:
  - Available for tight turns
  - Nose gear must be near centerline
  - Used for spot turns on deck

Visibility:
  - Good forward visibility (high cockpit)
  - Limited to sides (engines/gloves)
  - RIO can assist (intercom)
```

### Deck Handling Considerations

```
Taxi Speed:
  - Normal: 5-10 knots
  - Tight spaces: <5 knots
  - Deck personnel nearby: <3 knots

Deck Clearances:
  - Wingspan: 64 ft (unswept) - watch wing tips!
  - Tail height: ~16 ft - watch overhead
  - Engine inlets: FOD hazard - watch debris

Jet Blast:
  - Danger area behind engines: 200+ feet at idle
  - Personnel clear before throttle increase
  - Deck edge personnel: Watch jet blast direction
```

## Emergency Procedures

### Emergency Gear Extension

```
Indications:
  - Gear handle DOWN, no down-and-locked lights
  - Hydraulic failure (both systems)
  - Abnormal gear extension time

Procedure:
  1. Hydraulic pressure: CHECK (both systems)
  2. If pressure OK: Cycle gear (try again)
  3. If pressure LOW:
     a. Emergency gear extend handle: PULL
     b. Blow-down bottles fire
     c. Gravity drop + pneumatic assist
     d. Wait 10-15 seconds
  4. Check gear position indicators
  5. If still unsafe:
     - Perform fly-by (tower confirms position)
     - High-speed pass if necessary
     - Land with indicated down gear only
```

### Gear-Up Landing (All Gear Unsafe)

```
Procedure:
  1. Declare emergency
  2. Burn down fuel (reduce weight)
  3. Divert to field (not carrier!)
  4. Emergency blow-down: ATTEMPT
  5. If still unsafe:
     a. Approach at normal speed + 10 knots
     b. Delay touchdown as long as possible
     c. Touch down gently on centerline
     d. Cut throttles at touchdown
     e. Ride it out (aircraft will slide)
     f. Possible ejection if fire/uncontrollable

Result:
  - Extensive airframe damage
  - Aircraft usually total loss
  - Crew survivable if proper technique
```

### Single Main Gear

```
Indications:
  - One main gear down-and-locked
  - Other main gear unsafe

Procedure:
  1. Verify unsafe gear (fly-by, indicator)
  2. Attempt emergency blow-down
  3. If still single main gear:
     a. Divert to long runway
     b. Approach normal speed
     c. Touch down on good gear
     d. Hold wing up as long as possible
     e. As wing drops, cut throttles
     f. Aircraft will veer toward unsafe gear
     g. Maintain directional control with rudder
     h. Possible ejection if fire/departure

Technique:
  - Gentle touchdown critical
  - Delay wing drop as long as possible
  - Anticipate veer direction
  - Prepare for ejection
```

### Nose Gear Failure

```
Indications:
  - Nose gear unsafe
  - Both main gear down-and-locked

Procedure:
  1. Emergency blow-down: ATTEMPT
  2. If still unsafe:
     a. Approach normal speed
     b. Touch down on main gear
     c. Hold nose off as long as possible
     d. As speed decreases, nose will drop
     e. Nose contacts runway
     f. Maintain directional control (rudder)
     g. Cut throttles when nose drops

Result:
  - Damage to nose radome
  - Possible propeller strike (if tip tanks)
  - Crew safe (common emergency)
```

### Brake Failure

```
Single Brake Failure:
  - Use good brake normally
  - Compensate with rudder for pull
  - Longer landing roll
  - Plan for extra runway

Dual Brake Failure:
  - Use aerodynamic braking (speedbrake)
  - Prepare for barrier engagement (carrier)
  - Request arresting gear (field)
  - Long rollout (5,000+ feet)
```

## System Interactions

### With Hydraulics

File: `Systems/f-14b-hydraulic.xml`

Landing gear requires:
- Combined hydraulic system pressure
- Normal operation: >1800 PSI
- Emergency: Blow-down bottles (pneumatic)

Loss of hydraulics:
- Gear may not extend normally
- Emergency extension required
- Doors may not operate properly

### With Catapult

File: `Systems/catapult.xml`

Catapult launch requires:
- Nose gear down and locked
- Weight-on-wheels (WOW) signal from nose gear
- Kneeling position: KNEEL (extended)
- Launch bar deployed (separate system)

If nose gear not down: Catapult will not apply force.

### With CADC

File: `Systems/f-14b-cadc.xml`

CADC monitors gear position:
```
Maneuvering flaps inhibited when:
  - Gear DOWN (gear/gear-cmd-norm ≠ 0)
  - Prevents flap deployment during landing config

Aircraft-on-ground flag:
  - Set when any gear WOW signal present
  - Used throughout systems
```

### With Electrical System

File: `Systems/f-14b-electrical.xml`

Gear position indicators require electrical power:
- Cockpit lights: Powered by essential bus
- Emergency: Battery backup available
- Loss of power: No indication, but gear operates mechanically

## Key Properties (Property Tree)

### Control Properties

```
gear/gear-cmd-norm                  (0 or 1) Gear up/down command
gear/gear-pos-norm                  (0-1) Actual gear position
fcs/gear-kneel-cmd                  (0, 1, 2) Kneel command
```

### Output Properties

```
gear/unit[0]/WOW                    (0 or 1) Nose gear weight-on-wheels
gear/unit[1]/WOW                    (0 or 1) Left main gear WOW
gear/unit[2]/WOW                    (0 or 1) Right main gear WOW
gear/unit[0]/compression-ft         (feet) Dynamic compression
gear/unit[0]/base-compression-ft    (feet) Kneel offset
gear/unit[0]/compression-adjusted-ft (feet) Total for display
gear/unit[0]/z-position             (inches) Visual model position
position/aircraft-on-ground         (0 or 1) Any gear on ground
```

### Strut Properties (Per Gear)

```
gear/unit[n]/compression-ft         Current compression (weight)
gear/unit[n]/compression-velocity-fps  Strut velocity
```

## Visual Model Integration

The 3D model reads gear properties to animate:

```
Gear Doors:
  - Driven by gear-pos-norm
  - Open before gear extends
  - Close after gear retracts

Gear Struts:
  - Driven by gear-pos-norm
  - Linear extension/retraction

Gear Compression:
  - Driven by compression-ft
  - Strut compresses with weight
  - Dynamic during taxi/landing

Nose Gear Position:
  - Driven by z-position
  - Accounts for kneeling
  - Real-time update (smooth)
```

## Operational Procedures

### Normal Extension (Before Landing)

```
1. Speed: Below 280 KIAS
2. Gear Handle: DOWN
3. Wait for three green lights (down and locked)
4. If no lights within 10 seconds: Troubleshoot
5. Verify gear down visually (mirrors or wingman)
```

### Normal Retraction (After Takeoff)

```
1. Positive rate of climb (VSI positive)
2. Gear Handle: UP
3. Wait for three red lights (up and locked)
4. If no lights within 15 seconds: Troubleshoot
```

### Kneeling for Catapult

```
1. Taxi into catapult position
2. Hold brakes (parking brake ON)
3. Kneel Command: KNEEL (2)
4. Wait 5 seconds for full extension
5. Verify nose lowered (visual, crew signal)
6. Launch bar: EXTEND
7. Catapult crew: Hook up and tension
```

### Deck Storage (High Position)

```
1. Aircraft chocked and chained
2. Engines: SHUTDOWN
3. Hydraulics: OFF (APU may power for kneel)
4. Kneel Command: HIGH (0)
5. Wait 5 seconds for compression
6. Verify nose raised (visual check)
7. Prepare for deck storage or hangar
```

## Maintenance Considerations

### Inspection Points

```
Preflight:
  - Struts: Fluid level, leaks, extension
  - Tires: Pressure, wear, FOD damage
  - Brakes: Wear indicators, hydraulic leaks
  - Doors: Alignment, actuators, uplock/downlock
  - Steering: Shimmy damper, linkages
  - Kneeling: Actuator, fluid level, operation

Postflight:
  - Inspect for hard landing damage (compression)
  - Check tire wear (skid marks)
  - Inspect brake pads (heat, wear)
  - Look for hydraulic leaks
```

### Common Failure Modes

```
Slow Extension:
  - Low hydraulic pressure
  - Air in system
  - Worn actuator seals
  - Uplock not releasing

Shimmy (Nosewheel):
  - Worn shimmy damper
  - Low tire pressure
  - Worn bearings
  - Out-of-balance wheel

Brake Fade:
  - Overheated brakes
  - Worn brake pads
  - Hydraulic leak
  - Trapped air in lines

Kneeling Failure:
  - Actuator malfunction
  - Low hydraulic pressure
  - Position sensor failure
  - Mechanical jam
```

## References

1. **NAVAIR 01-F14AAD-1**: F-14A Aircraft Descriptive Data, Landing Gear Section
2. **F-14 NATOPS Flight Manual**: Landing gear operating procedures
3. **Implementation**: Richard Harrison, f-14b-landing-gear.xml
4. **JSBSim Documentation**: Landing gear contact points and ground reactions

## Related Documentation

- **HYDRAULICS.md**: Hydraulic power for gear operation
- **CATAPULT.md**: Catapult operations requiring kneeling system
- **FCS.md**: Ground/air mode logic from WOW signals
- **CADC.md**: Gear position interlocks with CADC

---

*The F-14 landing gear system exemplifies robust carrier aircraft design, with the unique kneeling feature enabling efficient carrier operations. This simulation captures the essential functionality while maintaining real-time performance.*
