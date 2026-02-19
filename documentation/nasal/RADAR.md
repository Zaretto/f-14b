# Nasal/Radar — AWG-9 Radar System

The `Nasal/Radar/` subdirectory implements the Westinghouse AWG-9 airborne fire control radar system.

## Files

| File | Lines | Purpose |
|------|-------|---------|
| awg-9.nas | ~2700 | AWG-9 radar modes, displays, and RWR |
| radar-system.nas | ~2400 | Core radar framework: contacts, scanning, terrain checking |
| radar-system-database.nas | ~270 | Aircraft/threat property database |
| rcs.nas | ~200 | Radar cross-section calculation and detection |
| rcs_mand.nas | ~125 | OPRF-specific RCS values |

## Architecture

```
radar-system-database.nas  (Aircraft/threat properties)
         │
         ▼
radar-system.nas           (Core framework)
  ├─ AIToNasal             reads /ai/models → emits AINotification
  ├─ NoseRadar             filters contacts to field-of-regard → emits FORNotification
  ├─ AIContact             individual target state
  ├─ Blep                  radar return (blip)
  ├─ OmniRadar             omnidirectional (RWR)
  └─ TerrainChecker        terrain occlusion via picking
         │
         ▼
rcs.nas + rcs_mand.nas     (RCS calculation)
  └─ inRadarRange()        distance/RCS visibility check
         │
         ▼
awg-9.nas                  (AWG-9 implementation)
  ├─ AirborneRadar         base class
  ├─ AWG9                  root system (CRM + ACM modes)
  ├─ RadarMode             mode base class
  ├─ RWR                   threat detection
  └─ xmlDisplays           cockpit display rendering
```

## Data Flow

1. `AIToNasal` reads the FlightGear AI property tree and emits an `AINotification` with all contacts
2. `NoseRadar` receives the notification, filters contacts into the field-of-regard frustum, and emits a `FORNotification`
3. `AirborneRadar` (AWG9) receives the `FORNotification` with the contact vector
4. Each contact is checked via `rcs.inRadarRange()` using the RCS database
5. Visible contacts become `Blep` objects (radar returns) with position, velocity, and RCS data
6. The active `RadarMode` processes bleps into display symbology
7. `xmlDisplays` renders the data to cockpit instruments
8. `RWR` (OmniRadar) separately detects all emitting radars via threat calculation

## Radar Modes

```
CRM (Range Mode)
├── RWS (Range While Search) ──→ STT (Single Target Track)
├── PD Search (Pulse Doppler) ──→ PD STT
├── P Search (Pulse) ──────────→ Pulse STT
└── TWS (Track While Scan)

ACM (Angle Mode)
└── PAL (Pilot Automatic Lockon)
```

### Mode Details

| Mode | Range | Azimuth | Bars | Notes |
|------|-------|---------|------|-------|
| RWS | 5-200 nm | 10-60° | 1-4 | Doppler search, primary search mode |
| PD Search | 5-200 nm | 10-60° | 1-4 | Pulse Doppler search |
| P Search | 5-200 nm | 10-60° | 1-4 | Mono-pulse, surface/marine detection |
| TWS | 20-80 nm | 20-40° | 2-4 | Tracks up to 10 targets simultaneously |
| PAL | short | narrow | — | ACM automatic target acquisition |
| STT | — | 0.22° | — | Hard lock, painter mode |

## AWG-9 Characteristics

- Field of regard: +/-60 degrees azimuth, +/-60 degrees elevation (minimum -80 degrees)
- Instant FoV radius: 1.1 degrees (0.5 bars worth)
- RCS reference: 89 nm at 3.2 m squared baseline
- Scan speed: 40-70 deg/sec depending on mode
- Blep fade time: 13 seconds
- Chaff filtering: 60% effectiveness

## Key Properties

| Property | Purpose |
|----------|---------|
| `controls/radar/elevation-deg` | Antenna tilt knob (+/-60 degrees) |
| `controls/radar/azimuth-deg` | Antenna azimuth knob (+/-60 degrees) |
| `instrumentation/radar/antennae-deg` | Current antenna tilt |
| `instrumentation/radar/radar-standby` | Standby mode |
| `instrumentation/radar/serviceable` | Radar serviceable flag |
| `sim/model/f-14b/instrumentation/radar-awg-9/wcs-mode` | Weapon control system mode |
| `instrumentation/ecm/on-off` | ECM status for RWR |

## Contact Types

| Code | Type | Detection Criteria |
|------|------|--------------------|
| 0 | AIR | Speed > 60 kt or known helicopter model |
| 1 | MARINE | Ship model or very low altitude |
| 2 | SURFACE | Ground vehicle |
| 3 | ORDNANCE | Missile |
| 4 | POINT | Generic point |
| 5 | TERRASUNK | Terrain with no loaded aircraft |

## RCS Calculation

The radar cross-section varies with aspect angle:

- Front (0 degrees): base RCS
- Side (90 degrees): 2.5x base RCS
- Belly (underside): 3.5x base RCS
- Rear (180 degrees): 1.75x base RCS

Detection formula: `maxDetectionRange = radarRange / (radarRCS / targetRCS) ^ 0.25`

### RCS Database Samples

| Aircraft | Frontal RCS (m squared) |
|----------|------------------------|
| F-14B | 12 |
| F-15C | 10 |
| F-16 | 2 |
| Typhoon | 0.5 |
| F-35 | 0.0005 |
| F-22 | 0.001 |
| B-1B | 6 |
| AWACS (E-3) | 110 |

## Radar Warning Receiver

The RWR uses `OmniRadar` (omnidirectional detection) with threat sorting based on the radar-system-database. Each known emitter has:

- `rwrCode`: 2-character emitter identifier (e.g. "14" for F-14)
- `baseThreat`: threat function based on bearing deviation
- `killZone`: effective engagement range in nm
- `hasAirRadar`: whether the emitter has an air-to-air radar

Two threat levels are displayed on the pilot's RWR display.

## Emesary Integration

The radar system uses Emesary notifications for decoupled communication:

- `AINotification` / `FORNotification` / `OmniNotification`: vectors of contacts
- `SliceNotification`: radar slice scan requests (elevation, yaw, radii, distance, filters)
- `ContactNotification`: individual contact scan requests
- Notifications can be bridged across multiplayer via property tree serialisation

## Performance Optimisations

- Adaptive RCS refresh: cache results 1-10 seconds based on probability
- Chaff filtering at 60% effectiveness
- Blep fade timeout reduces stale data
- Separate timers: fast (scan), medium (0.25s), slow (0.75s)
- Terrain and ECM checks via picking (expensive, applied selectively)
- `PartitionProcessor` (from fallback/frame_utils.nas) distributes contact processing across frames
