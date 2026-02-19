# DualControl — Pilot/RIO Multiplayer

The `DualControl/` subdirectory implements multiplayer synchronisation between a pilot and RIO (Radar Intercept Officer) flying the F-14 as two separate aircraft models that communicate over the FlightGear multiplayer network.

## Files

| File | Purpose |
|------|---------|
| f-14b-dual-control.nas | Core multiplayer sync framework — property encoding/decoding and aliases |
| f14.nas | Weapons station selection and ejection sequence |

## Usage

- `--aircraft=f-14b` — Pilot aircraft; backseat RIO can join
- `--aircraft=f14b-bs` — Backseater mode for joining a pilot's F-14B

## Pilot vs RIO Roles

| Aspect | Pilot | RIO |
|--------|-------|-----|
| Aircraft model | f-14b.xml | f-14b-bs.xml |
| Primary controls | Flight, some weapons | Radar, weapons station select, ECM |
| Radar access | Read-only (locked when RIO present) | Full control (AWG-9 mode, range, antenna) |
| Station selection | Stations 1 and 8 (centreline/aft fuselage) | Stations 2-7 (wing pylons) via MP sync |
| Navigation data | Provides (altitude, heading) | Consumes via property aliases |
| Ejection | Triggers sequence for both crew | Ejected simultaneously (0.2s delay) |

## Synchronisation Architecture

The system uses FlightGear's `dual_control_tools` (DCT) encoder/decoder pattern over generic multiplayer properties.

### Data Channels

| Channel | MP Property | Type | Data |
|---------|-------------|------|------|
| bs_switches1 | `generic/int[0]` | Integer | Station selectors 2-7, radar standby (7 bits) |
| bs_TDM1 | `generic/string[0]` | String (TDM) | Radar range, WCS mode, station selectors 1 & 8, antenna knobs, BARS index, azimuth field |
| Target designation | `generic/string[11]` | String | Target callsign for radar lock sync |

### Sync Flow

```
RIO (copilot)                          Pilot
─────────────                          ─────
Property changes
    │
    ▼
SwitchEncoder ──→ int[0] ──→ MP ──→ SwitchDecoder
TDMEncoder ───→ string[0] ──→ MP ──→ TDMDecoder
                                        │
                                        ▼
                                   Local property updates

Pilot properties
    │
    ▼
Property aliases ──────────────→ RIO reads pilot's instruments
```

### Property Aliases (RIO sees pilot's data)

When the RIO connects, 18 properties are aliased from the pilot's property tree:

- **Navigation:** current view, altitude, altimeter setting, heading, magnetic heading
- **Radar:** AWG-9 brightness/power, radar display mode, sweep factor
- **Tactical:** TID (Tactical Information Display) brightness/power
- **ECM:** ECM on/off, mode, HSD on/off, HSD needle deflection
- **Navigation aids:** NAV radial select, radar range, radar standby

## Connection Lifecycle

### RIO Joins

1. `pilot_connect_copilot(copilot)` called on pilot's instance
2. AWG-9 radar controls locked for the pilot (prevents conflicts with RIO)
3. Property listener set on `string[11]` for target designation sync
4. Switch and TDM decoders initialised to receive RIO's data
5. `copilot_connect_pilot(pilot)` called on RIO's instance
6. Property aliases created so RIO sees pilot's instruments
7. Dual-target display initialised on RIO's TID

### RIO Leaves

1. `pilot_disconnect_copilot()` unlocks AWG-9 controls
2. Property listener removed
3. `copilot_disconnect_pilot()` removes all property aliases

## Weapons Station Selection

From `f14.nas`:

- 8 pylon stations mapped to weapons selectors
- Stations 1 and 8: 3-position switch (-1 / 0 / +1)
- Stations 2-7: 2-position switch (-1 / 0) — only up/neutral allowed
- `station_selector(n)` cycles through states
- `station_select(n, state)` sets a specific state directly

## Ejection Sequence

From `f14.nas`:

1. Safety check: returns if already ejecting (`f14/done == 1`)
2. Creates two `armament.AIM` objects as dummy ejection seats (pilot ID 11, RIO ID 12)
3. Calls `releaseAtNothing()` on both seats
4. Switches camera view to follow the pilot's ejection via AI model tracking (view 115)
5. RIO ejection fires 0.2 seconds after pilot via `eject2()` callback

## Key Design Patterns

- **Radar lock:** pilot cannot adjust radar parameters when RIO is connected, preventing conflicts
- **Station split:** RIO controls wing pylons (2-7), pilot controls centreline (1) and aft fuselage (8)
- **Property aliasing:** RIO does not duplicate data; uses aliases to the pilot's property tree for real-time instrument readings
- **TDM encoding:** multiple values packed into a single multiplayer string property using time-division multiplexing, minimising MP bandwidth
- **Delayed ejection:** RIO ejects 0.2s after pilot for visual sequence control
