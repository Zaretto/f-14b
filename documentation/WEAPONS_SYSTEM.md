# F-14 Weapons System Technical Documentation

Technical documentation for the F-14 weapons system implementation.

For operational guidance, see [WEAPONS.md](WEAPONS.md).
For general aircraft operations, see [HELP.md](HELP.md).

---

## Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Missile Guidance System (fox2.nas)](#missile-guidance-system-fox2nas)
- [Station Management](#station-management)
- [Fire Control System](#fire-control-system)
- [Weapon Types](#weapon-types)
- [Guidance Laws](#guidance-laws)
- [Countermeasures](#countermeasures)
- [Emesary Integration](#emesary-integration)
- [Technical References](#technical-references)

---

## Overview

The F-14 weapons system is built on a modular, aircraft-agnostic guidance and fire control framework. The system simulates realistic missile guidance, station management, and fire control with full multiplayer synchronization via Emesary.

**Key Features:**
- Physics-based missile flight modeling (6026 lines in fox2.nas)
- Support for 10+ guidance laws (PN, APN, TVM, CLOS, etc.)
- Advanced countermeasures (chaff/flare resistance modeling)
- RCS-based detection and seeker performance
- Dynamic Launch Zone (DLZ) calculations
- Emesary-based multiplayer synchronization
- Station management with damage modeling

**Primary Author:**
- **Nikolai V. Chr. (weapons system lead)** - 156 commits to weapons system

**Contributing Authors:**
- Alexis Bory (original F-14 weapons integration)
- Fabien Barbier (missile guidance contributions)
- Richard Harrison (station management architecture, F-14 integration)
- Justin Nicholson (guidance improvements)
- Axel Paccalin (closest range calculations)
- Colin Geniet (guidance law refinements)

**Additional Contributors:**
- David Culp, Vivian Meazza, M. Franz (derived concepts)

---

## Architecture

### System Layers

The weapons system consists of four major components:

1. **Missile Guidance (fox2.nas)**: Generic missile/bomb guidance engine
2. **Station Manager (station-manager.nas)**: Pylon and store management
3. **Fire Control (fire-control.nas)**: Weapon selection and firing logic
4. **F-14 Integration (weapons.nas)**: Aircraft-specific weapon systems

### Data Flow

```
User Input (trigger)
    → Fire Control (fire-control.nas)
        → Station Manager (select pylon/weapon)
            → AIM Object (fox2.nas)
                → Release & Guidance Loop
                    → Emesary Notifications (multiplayer sync)
                        → Damage System (damage.nas)
```

### Communication via Emesary

**Emesary** is a publish-subscribe notification system that enables communication between disparate classes without them knowing about each other. All weapon-related events use Emesary notifications for decoupling:

- `ArmamentNotification`: Missile launches, impacts, status
- `ArmamentInFlightNotification`: Missile telemetry during flight
- `GeoBridgedNotification`: Position data bridged across multiplayer
- `WeaponNotification`: Weapon selection changes
- `WeaponRequestNotification`: MFD requesting current weapon info

**How it works:**
1. A class (e.g., fire control) sends a notification via `emesary.GlobalTransmitter.NotifyAll()`
2. All registered recipients receive the notification via their `Receive()` method
3. Recipients that handle the notification type process it; others ignore it
4. Notifications can be bridged across multiplayer via property tree serialization

This allows:
- **Decoupling:** Systems don't need to know about each other
- **Multiplayer synchronization:** Notifications automatically bridged to other aircraft
- **RIO/Pilot coordination:** Both cockpits receive the same notifications in dual-control mode
- **Modularity:** Weapon systems shared between F-14, F-15, and other aircraft

For detailed Emesary documentation, see [EMESARY.md](EMESARY.md).

---

## Missile Guidance System (fox2.nas)

### Overview

**fox2.nas** is a comprehensive guided weapons simulation with 6026 lines of code implementing realistic missile/bomb physics and guidance.

**Supported Weapon Types:**
- Air-to-air missiles (AIM-9, AIM-7, AIM-54, AIM-120)
- Air-to-ground missiles (AGM-84, AGM-88, AGM-154, AGM-158)
- Guided bombs (GBU-series)
- Unguided rockets and bombs
- Cruise missiles
- Surface-to-air missiles (when used as SAM simulator)

### Core AIM Class

The `AIM` class represents a single guided weapon instance. Key responsibilities:

**1. Seeker Simulation**
- Field of view (FoV) modeling with gimbal limits
- Target acquisition and lock logic
- Angular rate tracking for heat seekers
- Rosette scan patterns (SEAM mode)
- Caged/uncaged modes
- Lock-on-after-launch (LOAL) support

**2. Physics Modeling**
- 6-DOF flight dynamics
- Stage-based propulsion (up to 3 stages, rocket or jet)
- Parasitic and induced drag calculations
- Weight reduction as fuel burns
- G-load limits based on dynamic pressure
- Vector thrust modeling

**3. Guidance Laws**
- **Pure Pursuit** (direct): Aim directly at target
- **Proportional Navigation** (PN): Lead proportional to closing rate
- **Augmented PN** (APN): PN with target acceleration compensation
- **Optimal PN** (OPN): Modern adaptive PN (AIM-9X)
- **Line of Sight** (LOS): Maintain constant bearing to target
- **Command Guidance** (command): Ground/ship radar commands
- **TVM** (Track-Via-Missile): Target illumination with missile feedback
- **CLOS** (Command Line of Sight): Wire/radio guided to LOS
- **Terrain Following**: Low-altitude cruise missile mode
- **GPS/Inertial**: Coordinate-based guidance

**4. Target Tracking**
- Aspect angle calculations (all-aspect vs rear-aspect)
- Line-of-sight rate computations
- Closest approach predictions
- Target classification (air/marine/ground/point)
- Multi-target engagement support

**5. Countermeasure Resistance**
- Chaff effectiveness modeling (resistance: 0-1)
- Flare effectiveness modeling (resistance: 0-1)
- Seeker re-acquisition after CM break
- Sun lock modeling for IR seekers
- Terrain masking checks

### Weapon Properties

Each weapon is defined by 80+ properties in the aircraft XML. Key categories:

**Detection & Firing:**
```
max-fire-range-nm:       Maximum FCS firing range
min-fire-range-nm:       Minimum lock range
FCS-field-deg:           Fire control system FoV
detect-range-nm:         Seeker detection range
seeker-field-deg:        Seeker gimbal limit
seeker-beam-width-deg:   Instantaneous FoV
```

**Guidance:**
```
guidance:                heat/radar/semi-radar/gps/laser/etc.
navigation:              PN/APN/OPN/direct/LOS/command
proportionality-constant: N' value (typically 3-6 for PN)
all-aspect:              Boolean (vs rear-aspect only)
seeker-angular-speed-dps: Max target angular rate
```

**Engine:**
```
thrust-lbf-stage-1/2/3:  Thrust per stage
stage-1/2/3-duration-sec: Burn time per stage
weight-fuel-lbm:         Fuel weight
stage-1/2/3-jet:         Jet (vs rocket) engine
```

**Aerodynamics:**
```
weight-launch-lbs:       Total launch weight
drag-coeff:              Base Cd
cross-section-sqft:      Reference area
max-g:                   Maximum G-load
min-speed-for-guiding-mach: Minimum guidance speed
```

**Warhead:**
```
weight-warhead-lbs:      Warhead mass
arming-time-sec:         Time to arm fuze
self-destruct-time-sec:  Self-destruct timer
max-report-distance:     Proximity fuze radius
```

### Flight Loop

The main guidance loop (`flight()` function) runs at frame rate:

1. **Update State**
   - Read current position/orientation from AI model
   - Calculate speed, acceleration, altitude AGL
   - Update fuel weight and mass

2. **Propulsion**
   - Check stage timers
   - Apply thrust (altitude/Mach compensated for jets)
   - Update drag (base + induced + plume effects)

3. **Seeker Logic**
   - Check if target in FoV
   - Calculate aspect angle and LOS rate
   - Apply countermeasures checks
   - Re-acquire if lost lock (if supported)

4. **Guidance**
   - Execute active guidance law
   - Calculate steering commands
   - Apply G-load and speed limits
   - Update pitch/heading/roll

5. **Navigation Modes**
   - Terrain following (cruise missiles)
   - Loft profiles (ballistic climb)
   - Terminal dive (final approach)
   - Sample guidance (Phoenix midcourse)

6. **Proximity Check**
   - Calculate distance to target
   - Predict closest approach time
   - Trigger detonation if within fuze radius
   - Self-destruct if timer expires

7. **Telemetry**
   - Send Emesary notifications (if datalink enabled)
   - Update DLZ calculations
   - Report status to fire control

### Guidance Law Details

**Proportional Navigation (PN)**

Standard for most A/A missiles. Acceleration command proportional to line-of-sight rate:

```
a_command = N' × V_closing × λ_dot
```

Where:
- N' = navigation constant (typically 3-5)
- V_closing = closing velocity
- λ_dot = line-of-sight angular rate

**Implementation:** `AIM.guide_pn()`

**Augmented PN (APN)**

Adds target acceleration compensation. Used by modern SAMs:

```
a_command = N' × V_closing × λ_dot + (N'/2) × a_target_perpendicular
```

**Implementation:** `AIM.guide_apn()`

**Command Guidance**

Missile steered to maintain launcher's line-of-sight to target. Used by older SAMs and ship-launched missiles.

**Implementation:** `AIM.guide_command()`

**TVM (Track-Via-Missile)**

Missile seeker provides target illumination to ground radar, which computes guidance commands. Combines benefits of semi-active and command guidance.

**Implementation:** `AIM.guide_tvm()`

---

## Station Management

### Station Class

The `Station` class (station-manager.nas) represents a single weapon station (pylon or internal).

**Station Properties:**
- `id`: Unique station identifier (0-9 for F-14)
- `name`: Human-readable name ("L Glv Inner", "Fus Phoenix")
- `position`: 3D coordinates in aircraft frame
- `sets`: Array of loadout configurations
- `weapons`: Array of AIM weapon objects (or nil when empty)

**Capabilities:**
- Load/unload weapon sets
- Track mass and drag changes
- Jettison weapons
- Fire weapons in sequence
- Report category (1-3, flight envelope restrictions)
- Enable/disable stations dynamically

### Loadout Sets

F-14 stations support multiple configurations (defined in `ext_stores.nas`):

**FAD Light:**
- 4× AIM-9 Sidewinder

**FAD (Fighter Air Defense):**
- 2× AIM-9 Sidewinder
- 2× AIM-7 Sparrow
- 2× AIM-54 Phoenix

**FAD Heavy:**
- 4× AIM-9 Sidewinder
- 2× AIM-7 Sparrow
- 6× AIM-54 Phoenix

**Bombcat:**
- MK-83 general purpose bombs

### Station Configuration (F-14)

| Station | Location | Capacity | Notes |
|---------|----------|----------|-------|
| 1       | Left glove, outer | AIM-9, AIM-7, AIM-54 | Dual AIM-9 capable |
| 2       | Left glove, inner | AIM-7, AIM-54 | Phoenix capable |
| 3       | Left tunnel | AIM-54, fuel tank | Palletized Phoenix |
| 4       | Left shoulder | AIM-54, fuel tank | Phoenix capable |
| 5       | Right shoulder | AIM-54, fuel tank | Phoenix capable |
| 6       | Right tunnel | AIM-54, fuel tank | Palletized Phoenix |
| 7       | Right glove, inner | AIM-7, AIM-54 | Phoenix capable |
| 8       | Right glove, outer | AIM-9, AIM-7, AIM-54 | Dual AIM-9 capable |

**Note:** Stations 3 (left fuel tank) and 7 (right fuel tank) are integrated into fuel system when external tanks mounted.

### Mass and Drag Calculations

Station manager updates FDM properties:

**Mass:**
```
total_mass = launcher_mass + weapons_mass
```

Updated in:
- `/payload/weight[n]/weight-lb` (for GUI)
- JSBSim pointmass nodes (for flight dynamics)

**Drag:**
```
drag_area = pylon_drag + weapon_drag
```

Updated in JSBSim external properties for each station.

---

## Fire Control System

### FireControl Class

The `FireControl` class (fire-control.nas) manages weapon selection and release sequencing.

**Core Functions:**

**1. Weapon Selection**
- Cycles through weapon types (GUN → SW → SP-PH → A/G)
- Selects appropriate pylons based on weapon type
- Orders pylons by fire priority
- Handles dual-rail AIM-9 stations

**2. Trigger Handling**
- Checks master arm state
- Verifies weapon ready
- Manages ripple fire sequencing
- Prevents duplicate triggers

**3. AIM-9 Seeker Control**
- Cage/uncage toggle
- Auto-uncage on lock
- SEAM scan mode
- Coolant management (limited supply)

**4. Bomb Release Modes**
- CCIP (Continuously Computed Impact Point)
- CCRP (Continuously Computed Release Point)
- Ripple fire (spacing and quantity)

**5. Mode Management**
- Track weapon inventory
- Update cockpit lights (MSL PREP, HOT TRIG)
- Coordinate with radar for target info
- Monitor pylonFailure (damage model)

### F-14 Weapon Integration

**Stick Selector (`weapons.nas`):**

| Position | Value | Selects |
|----------|-------|---------|
| OFF      | 0     | No weapon |
| GUN      | 1     | M61A1 Vulcan |
| SW       | 2     | AIM-9 Sidewinder |
| SP-PH    | 3     | AIM-7 Sparrow / AIM-54 Phoenix |
| A/G      | 4     | MK-83 bombs (when HUD in A/G mode) |

**Master Arm Switch:**

| Position | Value | Effect |
|----------|-------|--------|
| OFF      | 0     | All weapons safe |
| ARM      | 1     | Weapons hot |
| TRAIN    | 2     | Simulated firing (no release) |

**Sidewinder Cooling:**

AIM-9 seekers require cooling before lock:
- Coolant supply: 7200 seconds (2 hours)
- Cool-down time: 30 seconds
- MSL PREP light: OFF until cooled, ON when ready
- SW COOL light: ON when cooling, OFF when ready

---

## Weapon Types

### AIM-9M Sidewinder

**Type:** Short-range infrared air-to-air missile

**Specifications:**
- Weight: 190 lbs (launch), 25 lbs (warhead)
- Propulsion: Single-stage solid rocket, 5000 lbf, 2.2 sec burn
- Max speed: Mach 2.5
- Max range: 10 NM (optimal 3-6 NM)
- Max G: 30G
- Guidance: Proportional navigation (PN), heat-seeking

**Seeker:**
- FoV: 80° total cone
- Beam width: 2.5°
- Lock range: 10 NM (cooled), 5 NM (warm)
- Angular rate: 40°/s maximum
- All-aspect capable

**Features:**
- SEAM (Sidewinder Expanded Acquisition Mode) scan
- Cage/uncage with auto-uncage on lock
- Radar slaving (follows STT lock)
- Coolant supply (2 hours)

**Countermeasures:**
- Flare resistance: 0.85
- Sun lock angle: 30° (loses lock if sun within cone)

---

### AIM-7M Sparrow

**Type:** Medium-range semi-active radar air-to-air missile

**Specifications:**
- Weight: 510 lbs (launch), 88 lbs (warhead)
- Propulsion: Single-stage solid rocket + sustainer, 5000 lbf, 6 sec burn
- Max speed: Mach 4.0
- Max range: 30+ NM
- Max G: 25G
- Guidance: Proportional navigation (PN), semi-active radar

**Seeker:**
- Requires continuous radar illumination from launch platform
- Lock range: 30 NM
- Beam width: 8°
- All-aspect capable

**Features:**
- Sample guidance until terminal phase
- Will go ballistic if radar lock broken
- Can resume guidance if lock reacquired

**Countermeasures:**
- Chaff resistance: 0.75 (vulnerable to chaff)
- Flare resistance: 1.00 (immune to flares)

---

### AIM-54C Phoenix

**Type:** Long-range active radar air-to-air missile

**Specifications:**
- Weight: 1008 lbs (launch), 135 lbs (warhead)
- Propulsion: Dual-stage solid rocket, 10000 lbf + 1000 lbf sustainer
- Max speed: Mach 5.0
- Max range: 100+ NM
- Max G: 17G
- Guidance: Inertial/command midcourse, active radar terminal

**Flight Profile:**
1. **Launch to 8 NM:** Sample guidance (TWS updates from AWG-9)
2. **8 NM to 11 NM:** Semi-active radar (imitates AIM-7 for terminal)
3. **11 NM to impact:** Active radar seeker (pitbull mode)

**Seeker:**
- Active radar range: 11 NM
- Lock range: 100 NM (via sample guidance)
- Beam width: 10°
- All-aspect capable

**Features:**
- Loft profile (climbs to cruise altitude)
- "Maddog" mode (launch without lock, autonomous search)
- Multi-target engagement (TWS mode)
- Extended self-destruct time (300 seconds)

**Countermeasures:**
- Chaff resistance: 0.90 (hard to fool)
- Inertial midcourse makes it resistant to jamming

---

### M61A1 Vulcan Cannon

**Type:** Six-barrel 20mm rotary cannon

**Specifications:**
- Ammunition: 675 rounds (at startup)
- Rate of fire: 6000 rounds/min (100 rps)
- Muzzle velocity: 3400 fps
- Effective range: ~2 NM
- Tracer rounds modeled

**Implementation:**
- FlightGear submodels for bullets
- Impact smoke effects
- Sound synchronized to firing

---

### MK-83 General Purpose Bomb

**Type:** 1000 lb unguided gravity bomb

**Specifications:**
- Weight: 1000 lbs (total), 445 lbs (warhead)
- Guidance: None (CCIP aiming computed)
- Drag: High-drag configuration
- Arming time: 1.5 seconds

**Delivery:**
- Dive bombing with CCIP pipper
- HUD shows impact point
- Release cue crosses out when insufficient time to arm
- Optimal delivery: 40° dive, 400+ kts

---

## Guidance Laws

### Comparison Table

| Guidance Law | Type | Use Case | Advantages | Disadvantages |
|--------------|------|----------|------------|---------------|
| **Direct (Pure Pursuit)** | Pursuit | Gravity bombs, old missiles | Simple, low energy | Inefficient intercept |
| **PN** | Predictive | Modern A/A missiles | Efficient, reliable | Needs closing velocity |
| **APN** | Predictive | SAMs, modern A/A | Handles maneuvering targets | Complex, high G |
| **OPN** | Adaptive | AIM-9X, modern IR | Optimal energy usage | Computationally intensive |
| **LOS** | Geometric | Old missiles, rockets | Low acceleration needed | Collision course only |
| **Command** | Ground-directed | SAMs, old ship missiles | Offboard computing | Requires datalink |
| **TVM** | Hybrid | Advanced SAMs | Best of command + SARH | Complex ground station |
| **CLOS** | Manual | Wire-guided ATGMs | Simple, jam-proof | Requires constant steering |
| **GPS/Inertial** | Coordinate | Cruise missiles, smart bombs | Jam-resistant | Cannot track moving targets |

### Proportional Navigation Implementation

The PN implementation in fox2.nas:

```nasal
guide_pn: func {
    # Calculate line-of-sight (LOS) vector to target
    me.los = me.target_coord.apply_course_distance(
        me.coord.course_to(me.target_coord),
        me.coord.direct_distance_to(me.target_coord)
    );

    # Compute LOS rate (angular velocity)
    me.los_rate = (me.los - me.old_los) / me.dt;

    # Proportional Navigation: a = N' * Vc * λ_dot
    me.guidance_accel = me.pro_constant * me.closing_velocity * me.los_rate;

    # Apply G-limit
    if (me.guidance_accel > me.max_g_current * G) {
        me.guidance_accel = me.max_g_current * G;
    }

    # Convert to pitch/yaw commands
    me.pitch_command = me.guidance_accel.vertical / me.speed;
    me.yaw_command = me.guidance_accel.lateral / me.speed;
}
```

**Key Parameters:**
- `pro_constant` (N'): Typically 3-5 for A/A, higher for SAMs
- `closing_velocity`: Relative speed along LOS
- `los_rate` (λ_dot): Angular rate of LOS rotation
- `max_g_current`: Dynamic pressure and thrust dependent

---

## Countermeasures

### Chaff Modeling

**Mechanism:**
Chaff creates false radar returns that can break semi-active and active radar locks.

**Resistance Calculation:**
```
chaff_effectiveness = (1 - weapon.chaffResistance) * chaff_cloud_density
```

**Missile Types:**

| Missile | Chaff Resistance | Notes |
|---------|------------------|-------|
| AIM-7 Sparrow | 0.75 | Vulnerable to chaff |
| AIM-54 Phoenix | 0.90 | Hard to fool (inertial backup) |
| AIM-120 AMRAAM | 0.85 | Modern processing |
| SA-6 Gainful | 0.70 | Old generation SAM |

**Reacquisition:**
Some missiles (semi-radar) can resume guidance after chaff break:
- A/A missiles: 5-10 second delay
- SAMs: 2-5 second delay (if TVM/command guided)

### Flare Modeling

**Mechanism:**
Flares emit intense IR signature to decoy heat-seeking missiles.

**Resistance Calculation:**
```
flare_effectiveness = (1 - weapon.flareResistance) * flare_intensity / distance²
```

**Missile Types:**

| Missile | Flare Resistance | Notes |
|---------|------------------|-------|
| AIM-9B/D/E | 0.10 | Very vulnerable (old seeker) |
| AIM-9L | 0.60 | Improved seeker filter |
| AIM-9M | 0.85 | Counter-countermeasure capable |
| AIM-9X | 0.95 | Advanced imaging seeker |

**Seeker Filter:**
- Old IR seekers: `seeker_filter` = 0 (no filtering)
- Modern seekers: `seeker_filter` = 1-2 (reject flares)

**Sun Lock:**
Heat-seeking missiles lose lock if sun within `sun_lock` angle (typically 30°).

---

## Emesary Integration

### Notifications

**ArmamentNotification:**

Sent on major weapon events:

```nasal
ArmamentNotification.new("mhit", damage, typeID, callsign)
```

- `mhit`: Missile impact with damage
- `mlaunched`: Missile launched
- `mover`: Missile passed target (miss)
- `mcrater`: Bomb crater created
- `mnullping`: Periodic position update for tracking

**Fields:**
- `Flags`: Event type flags
- `UniqueIdentity`: Weapon unique ID
- `SecondaryKind`: Warhead type ID
- `Kind`: Weapon class (ordnance = 2)
- `Distance`: Impact distance from target
- `Position`: Geo.Coord of event

### Multiplayer Synchronization

**Bridge System:**

Emesary notifications are "bridged" across multiplayer using the MP property tree. The bridge serializes notifications into a single string property (`sim/multiplay/emesary/bridge[19]`) that is transmitted via FlightGear's multiplayer system.

**How bridging works:**
1. `OutgoingMPBridge` receives notifications via Emesary's normal mechanism
2. Serializes notification data to a compact string format
3. Writes to the MP property (default: 128 chars, configurable)
4. FlightGear transmits the property via UDP to other clients
5. `IncomingMPBridge` on other clients deserializes the string
6. Retransmits notification to local recipients

**Message Reliability:**
- Messages retransmitted for their lifetime (default: 10 seconds)
- Sequence IDs prevent duplicate processing
- Provides "near-reliable" delivery (high probability, not guaranteed)
- UDP nature means some messages may be lost

**Update Rate:**
- Configurable transmission frequency (default: 1 Hz)
- Message lifetime allows multiple retransmit attempts
- Position updates: Higher frequency as needed
- Event notifications: Queued and transmitted in order

For detailed Emesary documentation, see [EMESARY.md](EMESARY.md).

---

## Technical References

### Source Files

| File | Lines | Purpose |
|------|-------|---------|
| Nasal/fox2.nas | 6026 | Missile guidance and flight simulation |
| Nasal/station-manager.nas | ~1100 | Station and store management |
| Nasal/fire-control.nas | ~900 | Fire control logic |
| Nasal/weapons.nas | ~1500 | F-14 weapons integration |
| Nasal/ext_stores.nas | ~600 | External stores configuration |

### Key Classes and Functions

**fox2.nas:**
- `AIM.new()`: Create weapon instance
- `AIM.release()`: Launch weapon
- `AIM.flight()`: Main guidance loop (per frame)
- `AIM.guide_pn()`: Proportional navigation
- `AIM.guide_apn()`: Augmented PN
- `AIM.guide_command()`: Command guidance
- `AIM.check_fov()`: Seeker FoV check
- `AIM.checkForFlare()`: Flare countermeasure
- `AIM.checkForChaff()`: Chaff countermeasure

**station-manager.nas:**
- `Station.new()`: Create station
- `Station.loadSet()`: Load weapon set
- `Station.getWeapons()`: Get weapons array
- `Station.getMass()`: Get mass [weapons, pylon]
- `Station.releaseWeapon()`: Fire weapon
- `Station.jettisonWeapon()`: Emergency jettison

**fire-control.nas:**
- `FireControl.new()`: Create fire control instance
- `FireControl.trigger()`: Handle trigger press
- `FireControl.selectWeaponType()`: Cycle weapon selector
- `FireControl.selectNextPylon()`: Cycle pylon selection
- `FireControl.cage()`: Cage/uncage IR seeker

### Design Philosophy

**Modularity:**
- fox2.nas is aircraft-agnostic (shared between F-14, F-15, others)
- Station manager separates store management from aircraft logic
- Fire control isolates weapon selection from guidance

**Realism vs Performance:**
- Physics-based guidance (6-DOF flight model)
- Frame-distributed calculations to maintain FPS
- Simplified aerodynamics where full CFD impractical
- Emesary partitioning to avoid MP bandwidth saturation

**Extensibility:**
- New weapons added via XML properties only
- No code changes needed for new missile types
- Guidance laws reusable across weapons
- Station sets defined in data files

### Developer Notes

**Adding New Weapons:**

1. Create property definition in `f-14-common.xml`:
```xml
<armament>
  <aim-xyz>
    <long-name>AIM-XYZ Missile</long-name>
    <thrust-lbf-stage-1>10000</thrust-lbf-stage-1>
    <!-- ... 80+ properties ... -->
  </aim-xyz>
</armament>
```

2. Add to station sets in `ext_stores.nas`
3. Create 3D model and submodel XML
4. Test guidance parameters (see comments in fox2.nas line 20+)

**Testing Missiles:**

1. Set `DEBUG_FLIGHT = 1` in fox2.nas for flight telemetry
2. Launch at 40000 ft, Mach 2, head-on engagement
3. Verify max range matches literature (accounting for closing rate)
4. Test against maneuvering target (30G evasion)
5. Test countermeasures (chaff/flare effectiveness)
6. Check performance impact (maintain 25+ FPS)

---

## Known TODOs

From source code comments:

**fox2.nas:**
- Side wind compensation for rail heading offset
- Flare/chaff handling improvements (more sophisticated filtering)
- Seeker FOV pattern checking (volumetric scan patterns)
- High elevation math improvements (gimbal lock near zenith)
- Horizon stabilized uncaged mode (AIM-9X feature)

**fire-control.nas:**
- Multiple simultaneous weapon types (mixed loadout selection)
- Weapon health monitoring (damaged seekers, low coolant warnings)

**station-manager.nas:**
- External stores WOW requirement (prevent jettison on ground)
- Station failure modes (hung ordnance, electrical failure)

---

## Validation

### Testing Methodology

**Missile Performance Validation:**
1. Max range tests: Launch at optimal altitude/speed, measure range to target hit
2. Min range tests: Verify arming time prevents close-range detonation
3. G-load tests: Target performs maximum-rate turn, measure missile tracking
4. Energy management: Launch at low speed/altitude, verify kinematic limitations

**Countermeasure Validation:**
1. Chaff effectiveness: Measure lock break probability vs resistance value
2. Flare effectiveness: Test against multiple missile generations
3. Sun lock: Verify angle threshold and re-acquisition behavior

**Multiplayer Validation:**
1. Launch missile, verify other players see launch and flight
2. Impact weapon, verify damage transmitted via Emesary
3. Test with 50+ NM separation (bridge range limits)
4. Verify flare/chaff synchronized across players

**Performance Validation:**
1. Launch 6× AIM-54 simultaneously (maximum loadout)
2. Maintain 25+ FPS during flight
3. Monitor Nasal execution time (should be < 10ms per frame)

---

## License

GPL 2.0

---

For operational procedures and tactical employment, see [WEAPONS.md](WEAPONS.md).

For radar system documentation, see [RADAR.md](RADAR.md).

For complete aircraft documentation, see [README.md](README.md).
