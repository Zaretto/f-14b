# F-14 AWG-9 Radar System Documentation

Technical documentation for the AWG-9 radar simulation implementation.

For operational guidance, see [WEAPONS.md](WEAPONS.md).
For general aircraft operations, see [HELP.md](HELP.md).

---

## Contents

- [Overview](#overview)
- [Radar Architecture](#radar-architecture)
- [Target Detection](#target-detection)
- [RCS Calculations](#rcs-calculations)
- [Radar Modes](#radar-modes)
- [Terrain Masking](#terrain-masking)
- [RWR System](#rwr-system)
- [Performance Optimizations](#performance-optimizations)
- [Technical References](#technical-references)

---

## Overview

The F-14's AWG-9 (AN/AWG-9) radar is simulated using a modular Nasal-based system that models:
- Target detection based on Radar Cross Section (RCS)
- Doppler filtering for pulse-doppler modes
- Line-of-sight terrain masking
- Radar Warning Receiver (RWR) threat detection
- Track-While-Scan (TWS) capabilities

**Implementation:** Nasal/Radar/radar-system.nas (radar framework), Nasal/awg_9.nas (AWG-9 specific logic)

**Primary Authors:**
- Alexis Bory (original AN/APG-63 base, 2008-2012)
- Richard Harrison (radar architecture design, AWG-9 adaptation, radar framework)
- **Nikolai V. Chr (radar system lead)** - RCS calculations, terrain masking, datalink/IFF integration, RWR logic, Emesary multiplayer integration, chaff detection, performance optimizations, semi-active radar guidance

**Version:** radar-system.nas v10.1 (May 2023), awg_9.nas v2.8b (June 2018)

---

## Radar Architecture

### Modular Design

The radar system is structured in three layers:

1. **RadarSystem Layer** (radar-system.nas)
   - `AIToNasal`: Converts FlightGear AI property tree to Nasal contact vector
   - `NoseRadar`: Antenna scanning, elevation slicing, target filtering
   - `TerrainManager`: Line-of-sight terrain checking
   - `OmniRadar`: RWR threat detection (360° coverage)

2. **Aircraft Radar Layer** (awg_9.nas)
   - `rdr_loop()`: Main radar processing at 20Hz
   - `az_scan()`: Antenna azimuth scanning simulation
   - Target tracking and TWS logic
   - Radar mode management

3. **Data Layer**
   - `AIContact`: Individual contact object with position, velocity, RCS
   - `Target`: AWG-9-specific target wrapper with tracking state
   - RCS database (rcs.nas)

### Communication via Emesary

**Emesary** is a publish-subscribe notification system that enables communication between disparate classes without them knowing about each other. The radar system uses Emesary for decoupled communication:

```
AIToNasal discovers contacts
    ↓
Sends AINotification via emesary.GlobalTransmitter.NotifyAll()
    ↓
Recipients receive notification:
    - NoseRadar (processes for antenna scan)
    - OmniRadar (processes for RWR)
    - AWG-9 radar logic (processes for tracking)
    - Other systems (ignore if not relevant)
```

**Benefits:**
- **Decoupling:** Radar, weapons, and displays don't need references to each other
- **Extensibility:** New systems can listen to existing notifications without code changes
- **Multiplayer:** Notifications can be bridged across multiplayer (e.g., missile launch warnings)
- **Modularity:** Same radar code used in F-14, F-15, and other aircraft

For detailed Emesary documentation, see [EMESARY.md](EMESARY.md).

---

## Target Detection

### Contact Discovery

**AIToNasal** scans the FlightGear AI property tree (`/ai/models`) to discover contacts:

1. **Listener-based updates**: Triggered when AI models are added/removed
2. **Frame-distributed scanning**: Processes one contact per frame to avoid lag
3. **Position conversion**: Supports both ECEF (Earth-Centered Earth-Fixed) and GPS coordinates

**Filters applied during scan:**
- Invalid contacts (impact reports, ballistic impacts)
- Dual-control backseat aircraft (same callsign as ownship RIO)
- Non-detectable types (per RCS database `isDetectable` flag)

### Detection Range Formula

Detection range is calculated using the **radar range equation**:

```
detectionRange = radarMaxRange / (referenceRCS / targetRCS)^(1/4)
```

Where:
- `radarMaxRange`: Maximum detection range for reference RCS (typically 5 m²)
- `targetRCS`: Actual RCS of target based on aspect angle
- `referenceRCS`: Reference RCS (default 5 m² for most systems)

**AWG-9 Detection Ranges:**

| Radar Range Setting | Max Range (NM) | Target RCS = 3.2 m² | Target RCS = 0.5 m² |
|---------------------|----------------|---------------------|---------------------|
| 5                   | 5              | 4.4 NM              | 2.8 NM              |
| 10                  | 10             | 8.9 NM              | 5.6 NM              |
| 20                  | 20             | 17.8 NM             | 11.2 NM             |
| 50                  | 50             | 44.4 NM             | 28.0 NM             |
| 100                 | 100            | 89 NM               | 56 NM               |
| 200                 | 200            | 178 NM              | 112 NM              |

Reference: AWG-9 can detect a 3.2 m² target at 89 NM (range setting 100).

---

## RCS Calculations

### RCS Database

The simulation includes an extensive RCS database (`rcs.nas`) with values for 70+ aircraft types:

| Aircraft Type | Front RCS (m²) | Notes |
|---------------|----------------|-------|
| F-22 Raptor   | 0.001          | Actual: 0.0001 m² |
| F-35A/B/C     | 0.0005         | 5th generation stealth |
| F-16C         | 2.0            | Clean configuration |
| Su-27 / J-11A | 15.0           | Large fighter |
| B-2 Spirit    | 0.001          | Actual: 0.0001 m² |
| F/A-18C       | 3.5            | Later blocks: 1 m² |
| MiG-29        | 6.0            | Estimate |

**Default RCS:** 5 m² (if aircraft not in database)

### Aspect-Dependent RCS

RCS varies dramatically with viewing angle. The simulation calculates RCS based on:

1. **Horizontal aspect** (front/side/rear view in 2D plane)
2. **Vertical aspect** (top/bottom view)

**RCS Multipliers:**

```
Front RCS:  1.0×  (reference, lowest)
Side RCS:   2.5×
Rear RCS:   1.75×
Belly RCS:  3.5×  (highest due to flat surfaces)
```

**Algorithm** (`getRCS()` in rcs.nas):

1. Calculate vector from radar to target
2. Project onto target's orientation vectors (nose, top)
3. Compute horizontal RCS (0-180° from nose)
4. Compute vertical correction (belly vs side RCS)
5. Return final RCS value

**Example:** An F-16C (2 m² front RCS) viewed from the side at 30° above horizon:
- Horizontal RCS: 2.5 m² (side aspect)
- Vertical correction: ~2.8 m² (partial belly exposure)
- **Effective RCS: ~2.8 m²**

### Caching and Refresh

RCS calculations are expensive. The system uses probabilistic caching:

```
if (update_age < 1 sec):  return cached value (100%)
if (update_age > 10 sec): recalculate (100%)
if (1 sec < update_age < 10 sec): recalculate with 5% probability per frame
```

This reduces CPU load while maintaining acceptable accuracy for moving targets.

---

## Radar Modes

### Mode Descriptions

| Mode | Full Name | Type | Notchable | Description |
|------|-----------|------|-----------|-------------|
| **PD SEARCH** | Pulse Doppler Search | Search | Yes | High sensitivity; shows closure rate; filtering rejects low-doppler returns |
| **RWS** | Range While Search | Search | Yes | Medium sensitivity; less doppler filtering |
| **P SEARCH** | Pulse Search | Search | No | No doppler filtering; ground/sea clutter visible |
| **PAL** | Pilot Auto Lock | Acq | - | 10 NM auto-STT when target centered |
| **PD STT** | Pulse Doppler STT | Track | Yes | Single Target Track with doppler filtering |
| **STT** | Single Target Track | Track | No | Pure pulse mode; immune to notching |
| **TWS AUTO** | Track While Scan Auto | Track | - | Tracks up to 18 targets; automatic priority |
| **TWS MAN** | Track While Scan Manual | Track | - | Manual target selection |

### Doppler Filtering (Notching)

**Pulse Doppler modes** (PD SEARCH, PD STT, RWS) reject targets with low closure rates.

**Notch threshold:**
```
notchSpeed = 100 kt (51.4 m/s)
```

A target is "notched" if:
```
|closureRate| < notchSpeed AND inGroundClutter
```

Where ground clutter exists when:
```
(targetAltitude - terrainAltitude) < clutter_altitude_threshold
```

**Defeating notch:**
- Use **P SEARCH** or **STT** mode (no doppler filtering)
- Change geometry to increase closure rate
- Wait for target to maneuver

---

## Terrain Masking

### Line-of-Sight Checks

The `TerrainManager` class uses FlightGear's terrain elevation API to check visibility:

**Algorithm** (`IsVisible()` method):

1. Get radar altitude and target altitude
2. Sample terrain elevation at 5 points along line-of-sight
3. For each sample point, check if terrain blocks the beam:
   ```
   if (sampleTerrainAlt > interpolatedBeamAlt):
       return FALSE  # target masked
   ```
4. If all samples clear, target is visible

**Sampling strategy:**
- Close targets (< 10 NM): 5 evenly-spaced samples
- Distant targets: Weighted toward midpoint (Earth curvature effects)

**Performance:** Terrain checks are expensive. Results are cached per contact with periodic refresh.

### Earth Curvature

For long-range detection (> 50 NM), Earth curvature affects line-of-sight:

```
horizonDistance_nm = 1.23 × sqrt(radarAltitude_ft)
```

**Example:** At 30,000 ft altitude:
- Radar horizon: ~213 NM
- Low-altitude targets masked beyond horizon

The simulation accounts for this in terrain sampling geometry.

---

## RWR System

### Threat Detection

The **Radar Warning Receiver** (RWR) is implemented as an omnidirectional radar (`OmniRadar` class).

**RWR detects:**
- Aircraft radar emissions (when their radar is enabled)
- SAM (Surface-to-Air Missile) sites
- AAA (Anti-Aircraft Artillery) radars

**Threat levels:**

| Level | Description | RWR Display |
|-------|-------------|-------------|
| 0     | No threat   | Not shown |
| 1     | Search radar illuminating | Threat symbol |
| 2     | Missile launch/STT lock | Priority threat symbol |

### RWR Processing

The `compute_rwr()` function updates the RWR display (TID in ECM mode):

1. Scan all contacts in 360° sphere
2. For each contact:
   - Check if it has an active radar (property `ai/models/.../radar/in-range`)
   - Determine threat level based on radar mode
   - Calculate bearing and range
3. Display top threats on TEWS (Tactical Electronic Warfare System) panel

**Update rate:** 5 Hz (every 0.2 seconds)

---

## Performance Optimizations

### Frame Distribution

The radar system distributes expensive operations across multiple frames:

**AIToNasal contact scanning:**
- Scans one contact per frame
- Triggered by listeners on `/ai/models/model-added` and `model-removed`
- Prevents multi-second stalls when many aircraft spawn

**Partitioned radar scanning:**

The AWG-9 processes targets in groups to maintain framerate:

```
maxTargetsPerFrame = 10
currentPartition = (frameCount % numPartitions)
```

Each frame processes 10 targets from a different partition. For 18 tracked targets, this means:
- Partition 0: targets 0-9
- Partition 1: targets 10-17

**Result:** Each target updated every 2-3 frames instead of every frame.

### Caching Strategies

**RCS calculations:**
- Cached with probabilistic refresh (see [RCS Calculations](#caching-and-refresh))
- 90-95% cache hit rate reduces CPU by ~10×

**Terrain visibility:**
- Cached per contact with 1-10 second TTL
- Reduces terrain API calls by ~20×

**Property tree reads:**
- All AI properties read once per scan and stored in `AIContact` object
- Subsequent radar logic uses cached values

---

## Technical References

### Source Files

| File | Lines | Purpose |
|------|-------|---------|
| Nasal/awg_9.nas | 1905 | AWG-9 radar modes, scanning, targeting |
| Nasal/Radar/radar-system.nas | ~3000 | Radar framework, contact management, terrain |
| Nasal/Radar/rcs.nas | 198 | RCS database and aspect calculations |
| Nasal/Radar/radar-system-database.nas | - | Aircraft detection parameters |

### Key Functions

**radar-system.nas:**
- `AIToNasal.readTree()`: Scan AI property tree for contacts
- `NoseRadar.getActiveBleps()`: Get contacts in antenna beam
- `TerrainManager.IsVisible()`: Line-of-sight terrain check

**awg_9.nas:**
- `rdr_loop()`: Main radar loop (20 Hz)
- `az_scan()`: Antenna azimuth scanning
- `compute_rwr()`: RWR threat processing

**rcs.nas:**
- `getRCS()`: Calculate aspect-dependent RCS
- `targetRCSSignal()`: Determine if target is within detection range
- `inRadarRange()`: Cached RCS check with probabilistic refresh

### Implementation Notes

**Coordinate Systems:**
- ECEF (Earth-Centered Earth-Fixed): Used for missile guidance
- GPS (Latitude/Longitude/Altitude): Used for AI models
- Conversion handled transparently in `AIContact.init()`

**Multiplayer Compatibility:**
- Radar state can be transmitted via Emesary notifications with MP bridging
- RIO backseat in dual-control mode receives full radar picture via local Emesary
- Missile launches synchronized via `ArmamentNotification` (bridged across MP)
- RWR warnings transmitted via `ArmamentNotification` when aircraft locks target

**FlightGear Core Dependencies:**
- AI property tree (`/ai/models`)
- Terrain elevation API (`geo.elevation()`)
- Coordinate utilities (`geo.Coord`)

---

## Radar Ranges Reference

### AWG-9 Specifications

**Antenna:**
- Azimuth scan: ±65° from nose
- Elevation coverage: -15° to +60°
- Scan patterns: Bar scan (1, 2, 4 bars), TWS scan

**Performance:**

| Specification | Value |
|---------------|-------|
| Maximum range | 200 NM (range setting) |
| Detection range (5 m² target) | 100+ NM |
| Detection range (3.2 m² target) | 89 NM |
| Track capacity | 18 targets (TWS) |
| STT lock capacity | 1 target |
| Update rate | 20 Hz (radar loop) |
| Antenna scan rate | ~60°/sec (mode dependent) |

### Missile Illumination

**AIM-54 Phoenix:**
- TWS mode: Inertial midcourse guidance (no illumination required)
- Terminal: Active radar seeker (onboard radar)
- Radar provides initial target data and updates

**AIM-7 Sparrow:**
- Requires continuous radar illumination
- Must maintain STT lock until impact
- Breaking lock causes missile to go ballistic

**AIM-9 Sidewinder:**
- Infrared guidance (no radar required)
- Radar can slave seeker to locked target

---

## Future Development

### Known TODOs (from source code)

**awg_9.nas:**
- Scan visibility check interval optimization
- Sweep line visual glitch when changing azimuth field
- RWR alert display override
- Chaff detection refinement
- Partitioned scanning improvements
- Vectorized field implementation
- Contact scanning method rework

**radar-system.nas:**
- Optimize slice requests when in STT mode
- Improve terrain checking for very long ranges (> 150 NM)
- Reduce property tree reads for contacts outside radar range

**Performance targets:**
- Maintain 60 FPS with 50+ AI contacts
- Reduce CPU usage of terrain checks by 50%
- Support 24-target TWS mode (F-14D upgrade)

---

## Validation

### Testing Methodology

**RCS validation:**
1. Compare calculated RCS vs published data for known aircraft
2. Verify aspect angle calculations using vector math
3. Test edge cases (directly above, directly behind)

**Detection range validation:**
1. Measure detection range for various RCS targets
2. Compare against radar range equation
3. Verify 89 NM detection for 3.2 m² reference

**Terrain masking validation:**
1. Test with known terrain (mountains, valleys)
2. Verify targets disappear/reappear correctly
3. Check performance with high terrain resolution

**Multiplayer validation:**
1. Verify radar picture matches between pilot/RIO
2. Test missile launches synchronized via Emesary
3. Confirm RWR displays other aircraft radars

---

## License

GPL 2.0

---

For operational procedures and tactical employment, see [WEAPONS.md](WEAPONS.md).

For complete aircraft documentation, see [README.md](README.md).
