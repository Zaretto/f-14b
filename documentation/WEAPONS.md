# F-14 Tomcat Weapons Operations

This guide covers the operation of the F-14's weapons systems and radar modes.

For general aircraft operations, see [HELP.md](HELP.md).
For technical documentation, see [README.md](README.md).

---

## Contents

- [Weapons Overview](#weapons-overview)
- [M61A1 Vulcan Cannon](#m61a1-vulcan-cannon)
- [AIM-9 Sidewinder](#aim-9-sidewinder)
- [AIM-7 Sparrow](#aim-7-sparrow)
- [AIM-54 Phoenix](#aim-54-phoenix)
- [MK-83 Bombs](#mk-83-bombs)
- [Radar Modes](#radar-modes)
- [Keyboard Reference](#keyboard-reference)

---

## Weapons Overview

### Available Weapons

| Weapon | Type | Guidance | Range |
|--------|------|----------|-------|
| M61A1 Vulcan | 20mm cannon | None | ~2 NM |
| AIM-9M Sidewinder | Air-to-air missile | Infrared (heat-seeking) | 10 NM |
| AIM-7 Sparrow | Air-to-air missile | Semi-active radar | 30+ NM |
| AIM-54 Phoenix | Air-to-air missile | Active radar | 100+ NM |
| MK-83 | General purpose bomb | Unguided (CCIP) | N/A |

### Load Configurations

Select via menu: **Tomcat Controls > Fuel and Stores**

| Configuration | Loadout |
|---------------|---------|
| FAD Light | 4x AIM-9 |
| FAD | 2x AIM-9, 2x AIM-7, 2x AIM-54 |
| FAD Heavy | Full missile loadout |
| Bombcat | MK-83 bombs |

### Master Arm

Before any weapon can be fired, Master Arm must be enabled:

- **Ctrl-w** - Cycle through Master Arm modes (OFF → ARM → TRAIN)
- The "X" on the gun symbol in the HUD indicates Master Arm OFF or TRAIN mode
- Master Arm must show ARM for weapons release

---

## M61A1 Vulcan Cannon

The M61A1 is a six-barrel 20mm rotary cannon.

### Specifications

- **Ammunition:** 675 rounds at startup
- **Rate of fire:** 6,000 rounds/minute
- **Effective range:** ~2 NM

### Operation

1. Select HUD A/A mode (`:AHa` or cockpit switch)
2. Select GUN mode with weapon selector (`m`)
3. Enable Master Arm (`Ctrl-w`)
4. Maneuver to place pipper on target
5. Fire with `e`

### HUD Symbology

When in gun mode, the HUD displays:
- **Pipper** - Aim point
- **G symbol** - Gun selected, with remaining rounds (×100)
- **Closure rate scale** - Active when target locked in TWS AUTO

### Tips

- Disable SAS Roll (`Ctrl-k`) for more precise gunning control
- Lead the target based on closure rate
- Short bursts conserve ammunition
- Gun can be reloaded via menu: **Tomcat Controls > Reload**

---

## AIM-9 Sidewinder

The AIM-9M is an infrared-guided (heat-seeking) air-to-air missile.

### Specifications

- **Guidance:** Proportional Navigation (PN) infrared
- **Lock range:** Up to 10 NM
- **Optimal firing range:** 3-6 NM
- **Seeker FOV:** ~80° cone from aircraft datum

### Operation

1. Select HUD A/A mode (`:AHa`)
2. Select SW mode with weapon selector (`m`)
3. Enable Master Arm (`Ctrl-w`)
4. Listen for seeker tone:
   - **Low buzz** - Seeker searching
   - **Loud tone** - Target locked
5. Maneuver target within seeker cone
6. Fire with `e` when tone indicates lock

### Lock Methods

**Radar-assisted:**
- Obtain radar lock (STT or TWS mode)
- Sidewinder seeker will slave to radar target

**Boresight:**
- AIM-9 can acquire targets without radar lock
- Point aircraft at heat source within seeker FOV
- Wait for lock tone

### Tips

- Fire at 3-6 NM for best Pk (probability of kill)
- Center target on velocity vector for optimal geometry
- Missile explodes at closest approach; if >70m, continues without guidance
- Rear-aspect shots (from behind target) have highest success rate

---

## AIM-7 Sparrow

The AIM-7 is a semi-active radar homing (SARH) missile.

### Specifications

- **Guidance:** Semi-active radar (requires continuous illumination)
- **Range:** 30+ NM
- **Requires:** Radar lock maintained until impact

### Operation

1. Select HUD A/A mode (`:AHa`)
2. Select SP-PH mode with weapon selector (`m`)
3. Obtain radar lock:
   - Use `y` to cycle targets
   - Use `r` for STT lock
4. Enable Master Arm (`Ctrl-w`)
5. Fire with `e`
6. **Maintain radar lock until impact**

### Important Notes

- Unlike AIM-9, there is no audible seeker tone
- Breaking radar lock causes missile to go ballistic
- Can resume guidance if lock reacquired (with reduced Pk)
- More resistant to flares than AIM-9; less resistant to chaff

### Dynamic Launch Zone

The HUD displays a dynamic launch zone indicator:
- **Bottom zone** - No escape (high Pk)
- **Middle zone** - Optimistic (medium Pk)
- **Top zone** - No hit chance

---

## AIM-54 Phoenix

The AIM-54 is a long-range active radar homing missile, unique to the F-14.

### Specifications

- **Guidance:**
  - Midcourse: Inertial/command (sample guided)
  - Terminal: Active radar
- **Range:** 100+ NM
- **Can engage:** Multiple targets simultaneously (with AWG-9 in TWS)

### Operation

1. Select HUD A/A mode (`:AHa`)
2. Select SP-PH mode with weapon selector (`m`)
3. Obtain radar lock or select target in TWS
4. Enable Master Arm (`Ctrl-w`)
5. Fire with `e`

### "Maddog" Mode

The AIM-54 can be fired without a radar lock:
- Missile will activate seeker and search for targets autonomously
- Use when radar is jammed or for area denial
- Lower Pk than guided launch

### Flight Profile

1. **Launch** - Missile ejects and motor ignites
2. **Climb** - Missile climbs to cruise altitude
3. **Cruise** - Inertial guidance toward predicted intercept
4. **Terminal** - Active radar seeker acquires target
5. **Impact** - Proximity or contact fuze

### Tips

- Most effective against non-maneuvering targets at long range
- Self-destruct time extended for long-range engagements
- Can engage targets at different altitudes simultaneously
- Phoenix is harder to spoof due to inertial midcourse guidance

---

## MK-83 Bombs

The MK-83 is a 1,000 lb general-purpose unguided bomb.

### Specifications

- **Weight:** 1,000 lbs
- **Guidance:** None (CCIP computed)
- **Delivery:** Dive bombing

### Operation

1. Select HUD A/G mode (`:AHg`)
2. Set manual wing sweep to 55° (`>` key, hold)
3. Select bomb mode with weapon selector (`m`)
4. Enable Master Arm (`Ctrl-w`)
5. Enter dive (approximately 40°)
6. CCIP pipper shows computed impact point
7. Release with `e` when pipper crosses target

### CCIP (Continuously Computed Impact Point)

The HUD displays:
- **Impact pipper** - Where bomb will hit
- **Release cue** - Crossed out when insufficient time to arm

### Tips

- Steeper dive angles improve accuracy
- Release early rather than late (bombs fall short of pipper at low altitude)
- Wing sweep at 55° is required for proper handling
- Pull up immediately after release to avoid blast

---

## Radar Modes

The AWG-9 radar has multiple operating modes for different tactical situations.

### Search Modes

| Mode | Key | Description | Notchable |
|------|-----|-------------|-----------|
| PD SEARCH | Ctrl-n | Pulse Doppler; highest sensitivity; shows closure rate | Yes |
| RWS | Ctrl-n | Range While Search; medium sensitivity | Yes |
| P SEARCH | Ctrl-n | Pulse; ground/sea clutter rejection | No |
| PAL | - | Pilot Auto Lock; 10 NM scan, auto-STT | - |

### Track Modes

| Mode | Key | Description | Notchable |
|------|-----|-------------|-----------|
| STT | r | Single Target Track (Pulse) | No |
| PD STT | r | Single Target Track (Pulse Doppler) | Yes |
| TWS | Ctrl-n | Track While Scan; tracks multiple targets | - |

### Notching

"Notching" is a defensive maneuver where the target flies perpendicular to the radar (placing closure rate near zero). Pulse Doppler modes can be defeated by notching; Pulse modes cannot.

### Radar Controls

| Key | Function |
|-----|----------|
| Ctrl-n | Cycle radar modes |
| q | Radar standby toggle |
| y | Next target |
| Shift-y | Azimuth scan width |
| i / I | Tilt radar up/down |
| D | Elevation scan coverage |
| r | Enter STT mode |
| U | Undesignate (break lock) |
| d | Pilot auto-lockon |

### TID (Tactical Information Display)

In TWS mode, the RIO's TID shows:
- All tracked targets with heading indicators
- Hostile/friendly classification (with IFF)
- Datalink contacts from other aircraft

---

## Keyboard Reference

### Weapon Selection

| Key | Function |
|-----|----------|
| m | Cycle weapon selector (GUN → SW → SP-PH → A/G) |
| Shift-m | Cycle pylon selection |
| Ctrl-w | Cycle Master Arm (OFF → ARM → TRAIN) |
| e | Fire/release selected weapon |

### Radar Operation

| Key | Function |
|-----|----------|
| Ctrl-n | Cycle radar mode |
| q | Radar standby |
| y | Next target |
| Shift-y | Azimuth width |
| i / I | Radar tilt up/down |
| D | Elevation coverage |
| r | STT lock |
| U | Undesignate |
| d | Auto-lockon |
| Shift-E / Shift-R | Radar range decrease/increase |
| h | Cycle HSD mode (radar/compass/ECM) |

### Countermeasures

| Key | Function |
|-----|----------|
| Ctrl-q | Release chaff/flare |
| Ctrl-d | DLC/Chaff switch |

### HUD Modes

| Keys | Function |
|------|----------|
| :AHa | A/A (Air-to-Air) mode |
| :AHg | A/G (Air-to-Ground) mode |
| :AHc | Cruise mode |
| :AHl | Landing mode |
| :AHt | Takeoff mode |
| :AHs | Toggle HUD on/off |

---

## Multiplayer Notes

### Dual Control

When operating with a RIO (Radar Intercept Officer) in multiplayer dual control mode:
- Pilot radar controls may be disabled
- RIO operates AWG-9 and weapons selection
- Coordinate via voice communication

### Emesary System

Weapons effects are synchronized across multiplayer:
- Missile launches visible to all players
- Damage calculated and applied via Emesary
- Chaff/flare visible to other aircraft
- Impact craters persist when damage enabled

---

## References

- NAVAIR 01-F14AAP-1 (NATOPS Flight Manual)
- FlightGear Wiki: [Grumman F-14 Tomcat](https://wiki.flightgear.org/Grumman_F-14_Tomcat)
