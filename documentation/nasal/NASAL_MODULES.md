# Nasal Modules — Root Scripts

The `Nasal/` directory contains the root-level Nasal scripts that implement the F-14's aircraft systems, weapons, instruments, and multiplayer integration. Scripts are loaded in dependency order as defined in `f-14-common.xml`.

Subdirectory-specific documentation is covered separately:

- [RADAR.md](RADAR.md) — AWG-9 radar system (`Nasal/Radar/`)
- [DUAL_CONTROL.md](DUAL_CONTROL.md) — Pilot/RIO multiplayer (`DualControl/`)
- [FALLBACK.md](FALLBACK.md) — Compatibility layer (`Nasal/fallback/`)

## JSBSim vs YASim

The F-14 originally used YASim and was converted to JSBSim from 2014 onwards. Several Nasal modules contain dual-path code controlled by the `usingJSBSim` global variable (set in `f14_globals.nas`). When running JSBSim, the following modules are wholly or partially inactive because their functionality has moved into the JSBSim FDM:

| Module | JSBSim Behaviour |
|--------|-----------------|
| SAS.nas | `computeSAS()` returns immediately — pitch, roll, and yaw SAS are implemented in `f-14-fcs.xml`. Only the trim functions, SAS channel enable/disable toggles, and JSBSim damper property forwarding remain active. |
| spoilers.nas | `computeSpoilers()` reads JSBSim spoiler positions for display and returns — all spoiler logic (roll, DLC, ground spoilers) is in the FDM. Only the ground spoiler arming toggle writes to JSBSim. |
| flaps_computer.nas | `applyBrakes()` (dual-purpose brake override) returns immediately — braking logic is in the FDM. The manoeuvre flaps/slats computation uses different property nodes (`controls/flight/flaps` and `fdm/jsbsim/fcs/slat-cmd` instead of `flapscommand` and `slats`). |
| instruments.nas | `compute_drag()` returns immediately — drag calculation is in the FDM. |
| sweep_computer.nas | Fully active under both FDMs, but under YASim the sweep was display-only with no aerodynamic effect. Under JSBSim the sweep commands are written to `fdm/jsbsim/fcs/wing-sweep-cmd` and directly affect the aerodynamic model. |
| cadc.nas | Under YASim the APC was display-only. Under JSBSim it writes to `fdm/jsbsim/systems/apc/active` and controls the FDM throttle. |
| engines.nas | AICS ramp computation and compressor stall detection only run under JSBSim. Nozzle and EGT computation use different property paths per FDM. |
| electrics.nas | Under YASim a basic electrical model is set up at load time. Under JSBSim, generator and hydraulic switch states are forwarded to JSBSim FDM properties. |

## File Summary

| File | Lines | Purpose |
|------|-------|---------|
| f-14b.nas | ~893 | Main aircraft initialisation and frame update loop |
| fox2.nas | ~5700 | Missile and weapon guidance (AIM-9, AIM-7, AIM-54, bombs) |
| damage.nas | ~1800 | Damage model with warhead database |
| instruments.nas | ~1200 | Cockpit instrument outputs and TACAN/ILS |
| fuel-system.nas | ~1105 | Multi-tank fuel system with crossfeed and refuelling |
| fire-control.nas | ~1206 | Weapon firing logic, selection, and ripple bombing |
| station-manager.nas | ~1237 | Pylon/store management framework |
| weapons.nas | ~399 | F-14-specific weapon definitions and load configurations |
| pylons.nas | ~399 | Physical pylon definitions and station instantiation |
| afcs.nas | ~555 | Automatic Flight Control System (autopilot) |
| SAS.nas | ~222 | Stability Augmentation System |
| cadc.nas | ~112 | Approach Power Compensator |
| flaps_computer.nas | ~171 | Manoeuvre flaps/slats and dual brakes |
| sweep_computer.nas | ~242 | Variable wing sweep control |
| spoilers.nas | ~155 | Roll, DLC, and ground spoilers |
| gear.nas | ~129 | Landing gear, NWS, and launchbar |
| engines.nas | ~545 | Engine management (AICS, EGT, JFS, afterburner) |
| electrics.nas | ~223 | Electrical system and HUD power |
| hud.nas | ~73 | HUD rendering and intensity |
| ext_stores.nas | ~176 | External stores and jettison logic |
| emesary.nas | ~717 | Emesary publish-subscribe notification system |
| emesary_mp_bridge.nas | ~461 | Multiplayer notification bridge |
| AircraftEventNotification.nas | ~103 | Property synchronisation notification |
| ArmamentNotification.nas | ~303 | Armament event notifications |
| GeoBridgedTransmitter.nas | ~83 | MP bridge setup for armament notifications |
| M_exec.nas | ~104 | Real-time frame-based execution loop |
| M_frame_notification.nas | ~90 | Frame notification system |
| M_nearest_carrier.nas | ~112 | Nearest carrier finder |
| awg_9.nas | ~1905 | AWG-9 radar wrapper and target management |
| datalink.nas | ~682 | Datalink protocol for inter-aircraft data sharing |
| iff.nas | ~112 | Identification Friend or Foe |
| gci-listener.nas | ~258 | Ground Controlled Intercept listener |
| an-arc-159v1.nas | ~230 | AN/ARC-159 UHF radio |
| an-arc-182v.nas | ~408 | AN/ARC-182 VHF/UHF/NAV radio |
| back-seat-instruments.nas | ~157 | RIO backseat data import/export |
| crash-and-stress.nas | ~583 | Structural damage from impacts and wing loading |
| environment.nas | ~43 | Standard atmosphere calculations |
| f14_globals.nas | ~76 | Global variables for shared state |
| chronograph.nas | ~47 | Cockpit stopwatch |
| liveries.nas | ~6 | Livery system initialisation |
| bombable_integration.nas | ~55 | Bombable damage system integration |
| rcs.nas | ~80 | Radar cross-section database and calculation |
| tacview.nas | ~100 | Tacview ACMI flight recording |
| vector.nas | ~100 | 3D vector math library |

---

## Main Aircraft

### f-14b.nas — Aircraft Initialisation

Main aircraft initialisation and per-frame update loop. Creates the `F14_exec` class that registers with Emesary and distributes frame updates to all subsystems.

**Key functions:**

- `F14_exec` — main subsystem class; `update()` called every frame
- `timedMotions()` — animated surface deflections (spoilers, wing sweep, nozzles)
- `canopyswitch()`, `toggle_cockpit_views()` — cockpit interaction
- `quickstart()`, `cold_and_dark()` — initialisation presets
- `eject()`, `fixAirframe()` — ejection sequence and damage repair
- `set_flood_lighting_colour()` — dome and instrument lighting
- Splash vector animation system for multiplayer-visible effects

**Properties:** Surface positions (spoilers, elevators, ailerons, wing sweep, slats), multiplayer sync (flap positions, wing tear states), lighting (dome, instrument, red flood), engine nozzle positions.

### f14_globals.nas — Global Variables

Shared state variables used across multiple modules.

**Key variables:**

- `usingJSBSim` — FDM detection flag (`getprop("/sim/flight-model") == "jsb"`)
- `AutoSweep`, `WingSweep` — wing geometry control
- `Nozzle1Target`, `Nozzle2Target` — engine nozzle positions
- `LeftSpoilers`, `InnerLeftSpoilersTarget` — control surface targets
- `OldPitchInput`, `SASpitch`, `SASroll` — SAS tracking
- `CurrentMach`, `CurrentAlt`, `CurrentIAS`, `Alpha` — flight state

---

## Flight Control

### afcs.nas — Automatic Flight Control System

Implements the AFCS autopilot with attitude hold, altitude hold, heading/ground track modes, and route manager integration.

**Key functions:**

- `afcs_engage_toggle()` — master AP engagement
- `afcs_attitude_engage()` — pitch and roll hold
- `afcs_heading_engage()` — DG heading hold
- `afcs_groundtrack_engage()` — true heading hold
- `afcs_altitude_engage_toggle()` — two-step altitude mode (enable then engage)
- `routeManagerUpdate()` — waypoint turn distance/lead time calculation
- `afcs_route_manager_exec` — Emesary-based route manager handler

**Internal state:** `ap_lock_att` (0=off, 1=engaged, 2=temporarily overridden, 3=with altitude), heading and altitude lock flags.

**Depends on:** SAS.nas (checks SAS channel enables before AFCS engagement).

### SAS.nas — Stability Augmentation System

Flight control dampening and authority limiting with roll quadratic law, pitch smoothing/filtering, and control stick steering overrides for the autopilot.

**JSBSim note:** `computeSAS()` returns immediately — all SAS computation (pitch, roll, yaw damping) is implemented in the JSBSim FCS (`f-14-fcs.xml`). The Nasal SAS code only runs under YASim. The trim functions, SAS channel enable/disable toggles, and JSBSim damper property forwarding remain active under both FDMs.

**Key functions:**

- `computeSAS()` — main per-frame SAS computation (YASim only)
- `trimUp()`, `trimDown()` — elevator trim with airspeed compensation
- `econt_rudder_trim()` — rudder trim adjustment
- `update_steering_deadZ()` — control stick dead zone update

**Filter parameters (YASim only):**

- Pitch smoothing: exponential filter (factor 0.1), proportional gain -0.05, bias limits +/-0.2
- Roll: quadratic law with 400 kt breakpoint
- Rudder: smoothing factor 0.2
- Pitch authority limiting below 230 kt

### cadc.nas — Approach Power Compensator

Automatically controls throttle during approach to maintain AOA setpoint. Also manages DLC auto-disengagement.

**JSBSim note:** Under YASim the APC was display-only. Under JSBSim the APC writes to `fdm/jsbsim/systems/apc/active` and controls the FDM throttle schedule.

**Key functions:**

- `computeAPC()` — main APC computation; monitors throttle, gear, and engine state
- `toggleAPC()`, `APC_on()`, `APC_off()` — APC engagement control

**Disengagement conditions:** Throttle advanced to MIL (>=0.91), flaps retracted (<25 degrees), on ground, gear up, or engine out.

**Engagement requirements:** Airborne, gear down, both engines running.

### flaps_computer.nas — Flaps Computer

Manages manoeuvre flaps and slats deployment based on angle-of-attack and Mach number. Overrides FlightGear brakes with a dual-purpose system (wheel brakes on ground, speed brakes in air).

**JSBSim note:** The dual-purpose brake override (`applyBrakes`) returns immediately under JSBSim — braking logic is in the FDM. The flaps/slats computation uses different property nodes per FDM.

**Key parameters:**

- Manoeuvre slat deployment: 7.7 degrees to 10.5 degrees alpha
- Manoeuvre flap extension ratio: 0.286
- Mach cutoff for slats: 0.5 + altitude x 0.000011667 (altitude-dependent)

**Key functions:**

- `computeFlaps()` — auto-deploys manoeuvre flaps/slats based on alpha/Mach envelope
- `controls.applyBrakes()` — override for dual-purpose brake system (YASim only)

### sweep_computer.nas — Wing Sweep Computer

Variable wing sweep (20 degrees to 68 degrees) with automatic sweep based on Mach/altitude, manual override, and oversweep modes. Enforces flap/sweep interlocks.

**JSBSim note:** Under YASim the sweep was display-only with no aerodynamic effect. Under JSBSim the sweep commands are written to `fdm/jsbsim/fcs/wing-sweep-cmd` and directly affect the aerodynamic model via NASA windtunnel data tables.

**Sweep modes:**

| Mode | Description |
|------|-------------|
| 0 (Auto) | Sweep follows Mach (20 degrees at low Mach, 60 degrees at high Mach) |
| 1 (Manual) | Pilot manually commands sweep |
| 2 (Off) | Sweep held at current position |
| 3 (Emergency) | Reserved |
| 4 (Oversweep) | Full 68 degrees on ground only, with guard/cover interlock |

**Key functions:**

- `computeSweep()` — main sweep computation
- `set_sweep(mode)` — set sweep mode
- `toggleOversweep()` — toggle oversweep (ground only)

**Interlocks:** Cannot oversweep if auxiliary flaps deployed; cannot move wings until aux flaps retracted below 5%; cannot oversweep airborne.

**Sweep rate:** 2.0 degrees/sec.

### spoilers.nas — Spoiler System

Manages spoiler deployment for roll control, direct lift control (DLC), and ground spoilers.

**JSBSim note:** `computeSpoilers()` reads JSBSim spoiler positions for display and returns immediately — all spoiler logic (roll control, DLC, ground spoilers) is implemented in the JSBSim FCS. Only the ground spoiler arming toggle writes to JSBSim (`fdm/jsbsim/fcs/spoiler-ground-brake-armed`).

**Spoiler configurations (YASim computation):**

- **Roll control:** outer spoilers deflect opposite to aileron input
- **DLC:** inner spoilers provide vertical lift control independent of roll
- **Ground spoilers:** all spoilers deploy fully on landing (armed + on ground + throttle at idle)

**Wing sweep bias:** Spoiler effectiveness reduced at high sweep angles (diminished at 56 degrees+, zero at 68 degrees).

**Key functions:**

- `computeSpoilers()` — main spoiler computation (YASim computation, JSBSim read-only)
- `toggleDLC()` — toggle Direct Lift Control
- `toggleGroundSpoilers()` — toggle ground spoiler arm

---

## Aircraft Systems

### engines.nas — Engine Management

Manages dual engines including EGT calculation, AICS (Air Inlet Control System) ramps, nozzle/afterburner control, JFS (Jet Fuel Starter) startup/shutdown, and compressor stall detection.

**JSBSim note:** AICS ramp computation and compressor stall detection only run under JSBSim. Nozzle and EGT computation use different property paths per FDM (Rankine EGT from JSBSim vs degrees-F from YASim).

**Key functions:**

- `computeAICS()` — air inlet ramp control (3 ramps per engine, Mach-scheduled; JSBSim only)
- `computeNozzles()` — nozzle opening and EGT calculation
- `engineControls()` — JFS startup sequencing and state management
- `engine_crank_switch(n)` — engine start crank switch handler
- `fire_handle(n)` — fire handle toggle (fuel cutoff)

**AICS ramp schedule (JSBSim only):**

| Mach | Ramp Position |
|------|---------------|
| < 0.5 | All stowed |
| 0.5–1.2 | Linear interpolation |
| 1.2–2.0 | Supersonic scheduling, ramp2 opens for cooling |
| > 2.0 | Fully open |

**JFS startup sequence:**

1. Crank switch moved to engine (L or R)
2. If no bleed air available, JFS starts (10s to ready)
3. Once JFS running, starter engaged
4. Engine spins up (N1 rises)
5. When both engines running or sufficient time elapsed, crank switch resets
6. JFS automatically shuts down after 55s

**Afterburner:** 5-stage burner intensity model with nozzle opening proportional to burner state.

**Compressor stall (JSBSim only):** Monitors JSBSim P0-stall indicator (>0.98 = stall detected).

### fuel-system.nas — Fuel System

Complex multi-tank fuel system with realistic crossfeed, proportioners, and aerial refuelling.

**Key classes:**

- `Tank` — individual fuel tank with capacity and level management
- `Prop` — proportioner/feed line (distribution system)
- `Neg_g` — negative-G fuel check valve simulation
- `Valve` — generic fuel valve (dump valve)

**Tank structure (10 tanks):**

| Tank | Capacity (lbs) |
|------|----------------|
| FWD fuselage | 4700 |
| AFT fuselage | 4400 |
| Left/Right beam box | 1250 each |
| Left/Right sump | 300 each |
| Left/Right wing | 2000 each |
| Left/Right external | 2000 each |
| Left/Right feed (proportioners) | 10 each |

**Key functions:**

- `init_fuel_system()` — initialise tanks and proportioners
- `fuel_update()` — per-frame fuel transfer between tanks
- `calc_levels()` — calculate total fuel across system
- `set_fuel()` — set total fuel with distribution algorithm

### electrics.nas — Electrical System

Manages electrical system state including console lighting, HUD visibility, generator switches, emergency systems, and hydraulic transfer pump.

**JSBSim note:** Under YASim a basic electrical model is set up at load time. Under JSBSim, generator and hydraulic switch states are forwarded to JSBSim FDM properties (`fdm/jsbsim/systems/electrics/*`, `fdm/jsbsim/systems/hydraulics/*`).

**Key functions:**

- `set_console_lighting()` — console lighting based on DC main bus and panel brightness
- `runEMMISC()` — main electrical computation
- `master_caution_pressed()` — master caution reset
- `master_test_select_switch(n)` — master test panel cycling (positions 0–10)

**HUD visibility:** Requires AC left main bus > 5V AND HUD power switch ON.

**Generator control:** Left/right generator switches and emergency generator with guard interlock.

### gear.nas — Landing Gear

Landing gear control including nose wheel steering (NWS), gear up/down safety interlock, and launchbar animation for carrier operations.

**Key functions:**

- `computeNWS()` — nose wheel steering (active below 80 kt groundspeed)
- `controls.gearDown(v)` — gear control with WOW safety interlock (prevents retraction on ground)
- `update_launchbar()` — launchbar position for carrier catapult
- `ldg_hdl_main()` — gear handle animation

**Carrier integration:** Detects carrier linkage, manages JBD (Jet Blast Deflector) control, listens to launchbar state changes.

### hud.nas — Head-Up Display

HUD rendering, intensity control, and 3D symbol positioning.

**Key functions:**

- `update_hud()` — update HUD visibility and intensity based on view
- `develev_to_devroll(dev_rad, elev_rad)` — convert deviation/elevation angles to roll-corrected HUD screen coordinates

**Physical parameters:** Eye-to-HUD distance 0.46 m, HUD field-of-view radius 0.085 m.

**Intensity:** Base intensity from pilot knob, reduced by G-induced redout alpha. Only updates in cockpit view.

### chronograph.nas — Cockpit Stopwatch

Elapsed-time stopwatch for cockpit clock with start/stop/reset toggle logic.

### liveries.nas — Livery System

Minimal initialisation script that calls `aircraft.livery.init()` to load paint schemes from `Models/Liveries/`.

---

## Weapons and Stores

### fox2.nas — Missile and Weapon Guidance

Universal guided/cruise missile, rocket, and bomb guidance system. Implements the complete missile flight model from seeker acquisition through terminal guidance.

**Key class: `AIM`**

- `new()` — create weapon instance
- `start()`, `stop()` — seeker lock management
- `release()`, `releaseAtNothing()` — fire weapon
- `eject()` — drop without arming

**Guidance modes:** Radar (semi-active, active), heat-seeking, GPS, inertial, anti-radiation, proportional navigation, command-to-line-of-sight (CLOS).

**Seeker patterns:** Circle, rosette, double-D.

**Missile states:** STANDBY, SEARCH, LOCK, FLYING.

**Features:** Drag modelling, flight physics simulation, target acquisition and tracking, chaff/flare countermeasure response.

### fire-control.nas — Weapon Firing Logic

Central fire control system managing weapon selection, firing, and ripple bombing.

**Key class: `FireControl`**

- `trigger()` — process firing request
- `cycleWeapon()`, `cycleLoadedWeapon()` — weapon selection cycling
- `selectPylon()` — manual pylon selection
- `jettisonAll()`, `jettisonFuelAndAG()` — jettison logic
- `rippleFireStart()`, `rippleTest()` — ripple bombing implementation

**Features:** Dual weapon selection (matched pairs), ripple bombing with distance spacing, weapon/ammo counting per type, master arm integration, CCRP/CCIP drop mode support.

### station-manager.nas — Pylon/Store Management

Framework for managing all weapon and store stations with multiple station types.

**Key classes:**

| Class | Purpose |
|-------|---------|
| `Station` | Base station class |
| `InternalStation` | Fixed internal mounts (cannon) |
| `FixedStation` | Fixed external mounts (CFT tanks) |
| `Pylon` | Ejectable pylon with weapons/stores |
| `WPylon` | Variant without fuel tank handling |
| `SubModelWeapon` | Cannon/rocket submodel wrapper |
| `FuelTank` | External fuel tank implementation |
| `Submodel` | Generic model container |
| `Dummy` | Placeholder for non-functional items |

**Key methods:** `loadSet()` (mount weapon configuration), `fireWeapon()` (release weapon), `jettisonAll()` (drop all stores), `calculateMass()` (update point mass), `calculateFDM()` (update drag for JSBSim).

### weapons.nas — Weapon Definitions

F-14-specific weapon definitions, station mapping, and load configurations.

**Load configurations:** FAD (full air defence), FAD light, FAD heavy, Bombcat, clean, airshow.

**Key functions:**

- `weapons_init()` — initialise system
- `armament_update()` — per-frame weapon status (cooling lights, ready status)
- `fad()`, `fad_l()`, `fad_h()`, `bomb()`, `clean()`, `airshow()` — load configuration presets
- `refuelFull()`, `refuel50()` — refuelling presets

### pylons.nas — Pylon Definitions

Physical pylon instantiation with 3D positions, drag areas, and mass properties.

**Pylon configuration:**

| Pylon | Location | Primary Load |
|-------|----------|-------------|
| 1A/8A | Outboard | AIM-9 |
| 1B/8B | Inboard | AIM-7 / AIM-54 / AIM-9 |
| 2/7 | Mid-wing | Fuel tanks (267 gal) |
| 3–6 | Centre | AIM-7, AIM-54, or MK-83 |
| I | Internal | M61A1 cannon |

### ext_stores.nas — External Stores Management

External weapons/tank load configuration, pylon selection, ACM jettison logic, and drop tank animation.

**Key functions:**

- `ext_loads_set(s)` — set load configuration
- `emerg_jettison()` — emergency jettison (all A/G ordnance + fuel tanks + missiles, preserves AIM-9)
- `do_acm_jettison()` — ACM selective jettison (gear up, guard engaged, 2-second hold required)
- `droptanks(n)` — drop tank ground impact animation

---

## Emesary Framework

### emesary.nas — Notification System

Core inter-object publish-subscribe notification system enabling decoupled communication between aircraft systems without them knowing about each other.

**Key classes:**

- `Transmitter` — sends notifications to registered recipients
- `Notification` — base message class (supports `TypeId`, `IsDistinct`, serialisation)
- `Recipient` — base receiver class (implements `Receive()` method)
- `GlobalTransmitter` — global instance for aircraft-wide messaging
- `QueuedTransmitter` — queue-based variant for batch processing

**Encoding classes for MP transmission:**

- `BinaryAsciiTransfer` — base-248 encoding engine
- `TransferInt`, `TransferCoord`, `TransferString`, `TransferByte`, `TransferNorm`, `TransferFixedDouble`

### emesary_mp_bridge.nas — Multiplayer Bridge

Bridges emesary notifications across the multiplayer network using property tree encoding.

**Key classes:**

- `OutgoingMPBridge` — encodes and transmits notifications over MP
- `IncomingMPBridge` — decodes and processes incoming MP notifications

Handles message sequencing, expiry, and distinct vs non-distinct messages. Transmits on `/sim/multiplay/` bridge indices 17–19.

### AircraftEventNotification.nas — Property Sync

Property synchronisation notification for multiplayer bridging. Synchronises 60+ aircraft properties over MP including fuel, N1/N2, afterburner status, electrical buses, nav/comm frequencies, landing gear, flaps, control surfaces, and canopy.

### ArmamentNotification.nas — Armament Events

Armament event notifications for weapon impacts, in-flight tracking, and environmental damage.

**Key classes:**

- `ArmamentNotification` — hit notifications (location, bearing, distance, type)
- `ArmamentInFlightNotification` — missile/projectile tracking (position, heading, pitch, velocity, callsign)
- `StaticNotification` — environmental damage markers (craters, smoke)
- `ObjectInFlightNotification` — generic in-flight objects

### GeoBridgedTransmitter.nas — Armament MP Bridges

Sets up three MP bridges on indices 17–19 for armament and geo-event notifications:

| Bridge | Index | Notification Type | Frequency |
|--------|-------|-------------------|-----------|
| 17 | Objects | ObjectInFlightNotification | 5 Hz |
| 18 | Missiles | ArmamentInFlightNotification | 1.3 Hz |
| 19 | Hits | ArmamentNotification + StaticNotification | 0.67 Hz |

### M_exec.nas — Frame Executive

Real-time frame-based execution loop. Fires `frameNotification` at approximately 50 Hz (adaptive to frame rate). Monitors frame rate, elapsed time, and integrates with `notifications.frameNotification`.

### M_frame_notification.nas — Frame Notification

Frame-based notification system allowing modules to register for per-frame updates.

**Key classes:**

- `FrameNotification` — carries frame count, elapsed time, monitored properties
- `FrameNotificationAddProperty` — dynamic property registration

### M_nearest_carrier.nas — Nearest Carrier

Locates the nearest aircraft carrier via the AI model tree. Uses `PartitionProcessor` for efficient scanning. Writes result to `/sim/model/f-14b/tuned-carrier`.

---

## Radar and Sensors

### awg_9.nas — AWG-9 Radar Wrapper

High-level AWG-9 radar interface and target management. See [RADAR.md](RADAR.md) for the full radar system documentation.

**Key components:**

- `rdr_loop()` — main radar processing loop
- `az_scan()` — azimuth scanning with sweep animation
- `Target` — target object with bearing, range, closure rate calculations
- `TerrainManager` — line-of-sight check via terrain picking

**Radar modes:** PULSE_SRCH, PD_SRCH, TWS (Track While Scan), RWS (Range While Search), STT (Single Target Track), PAL (Pilot Automatic Lockon).

**Key variables:** `tgts_list` (visible targets), `completeList` (all detected), `active_u` (selected target for weapons), `WcsMode` (weapon control system mode).

### rcs.nas — Radar Cross-Section

RCS database and calculation for radar detection range. Contains RCS values by aircraft model (e.g. F-14B: 12 m squared, F-16: 2 m squared, B-1B: 6 m squared). Backseater variants marked as near-invisible (0.0001).

**Key function:** `test()` — RCS calculation based on aspect angle, target heading/pitch/roll.

---

## Communications and Datalink

### datalink.nas — Datalink Protocol

Inter-aircraft communication and data sharing protocol.

**Key components:**

- Channel hashing with time/callsign (10-minute periods)
- Extensions: Identifier, Contacts (IFF tracking), Point (coordinate broadcast)
- `Contact` class — represents remote aircraft with tracked/friendly status

**Properties:** Power and channel on `/instrumentation/datalink/`, data transmitted on MP string 7.

### iff.nas — Identification Friend or Foe

MD5-based channel hashing for friend/foe identification.

- `iff_hash` — generates time-based hashes from callsign + channel
- `interrogate()` — verifies friendly aircraft
- Refresh rate: 120 seconds
- Output on `/sim/multiplay/generic/string[4]`

### gci-listener.nas — Ground Controlled Intercept

Receives tactical picture updates from GCI models over multiplayer.

**Key functions:**

- `find_aew_cx()` — locates nearest AEW/GCI model
- `check_messages()` — decodes PICTURE, BOGEY DOPE, CUTOFF responses

**Message format:** Colon-separated strings with bearing, range, altitude, aspect.

### an-arc-159v1.nas — AN/ARC-159 UHF Radio

AN/ARC-159 UHF radio (225–400 MHz) with 20 preset channels.

**Modes:** OFF, MAIN, BOTH, DF. **Functions:** PRESET, MANUAL, GUARD.

Interfaces with `comm[1]` radio. State persisted via `aircraft.data`.

### an-arc-182v.nas — AN/ARC-182 VHF/UHF/NAV Radio

AN/ARC-182 combined VHF/UHF/NAV radio with 20 presets and TEST mode.

**Modes:** OFF, T/R, T/R+GUARD, DF, TEST. **Functions:** GUARD, MANUAL, G, PRESET, READ, LOAD.

Guard frequency: 243 MHz. Interfaces with `comm[0]` and `nav[0]`. State persisted via `aircraft.data`.

### back-seat-instruments.nas — RIO Instruments

RIO backseat data import/export over multiplayer for dual-control operation.

**Key functions:**

- `instruments_data_import()` — receives pilot data (IAS, Mach, fuel, TACAN, electrical)
- `instruments_data_export()` — sends RIO aircraft type to pilot
- `check_pilot_callsign()` — locates pilot MP node

Pilot data received via `/sim/multiplay/generic/string[1]`.

---

## Instruments and Environment

### instruments.nas — Cockpit Instruments

Cockpit instrument outputs and command handling including TACAN, ILS, and carrier approach.

**JSBSim note:** `compute_drag()` returns immediately — drag calculation is in the FDM.

**Key class:** `ARA63Recipient` — AN/SPN-46 carrier approach system receiver (Emesary-based).

**Key functions:**

- TACAN frequency and bearing calculation
- ILS glideslope deviation (0.75 mile at 360 ft standard)
- Magnetic deviation computation
- Carrier approach guidance signal processing (ACL, wave-off lights)

**Properties:** `instrumentation/tacan/*`, `instrumentation/nav/*`, carrier approach lights.

### environment.nas — Standard Atmosphere

Standard atmosphere calculations for density and speed of sound.

**Key function:** `rho_sndspeed(altitude)` — returns air density and speed of sound at a given altitude.

Used by radar horizon and RWR range calculations.

---

## Damage and Stress

### damage.nas — Damage Model

Damage model with warhead database (80+ warhead types) and aircraft failure modes.

**Key features:**

- Shell/projectile database (~16 cannon/rocket types with damage values)
- Damage calculation based on warhead mass and hit probability
- HP-based damage system (alternative to failure modes)
- Configuration: `full_damage_dist_m`, `hp_max`, `use_hitpoints_instead_of_failure_modes_bool`

**Properties:** `sam/damage` (HUD damage percentage), `payload/d-config/*`.

### crash-and-stress.nas — Structural Damage

Structural damage from impacts and wing loading stress.

**Key class:** `CrashAndStress` — monitors wing loading, impact forces, G-forces. Integrates with `FailureMgr` for cascading failures. Supports both JSBSim and YASim property paths.

**Key parameters:**

- `wingLoadLimitUpper` / `wingLoadLimitLower` — G-force thresholds
- Stress sound triggers: crack, creak, detach

### bombable_integration.nas — Bombable Integration

Optional integration with the Bombable damage system. Listens to `/bombable/attributes/damage` and triggers wing breaking at >70% damage via `f14.breakWing()`.

---

## Utilities

### vector.nas — Vector Math

3D vector math library for missile guidance, aircraft rotation, and coordinate transformations.

**Key object:** `Math` — provides Cartesian/Euler conversions, matrix rotations (roll/pitch/yaw), cross product, dot product, magnitude, Rodrigues' rotation formula, and vector angle/aspect calculations.

### tacview.nas — Tacview Recording

Tacview ACMI recording for post-flight mission playback and analysis.

**Key functions:**

- `startwrite()` — creates `.acmi` file with timestamp
- `mainloop()` — writes ownship and MP aircraft positions each frame

**Output:** `/Export/tacview-f14-YYYY-MM-DDTHH-MM-SS.acmi`. Thread-safe file I/O via mutex.

---

## System Relationships

```
M_exec.nas (rtExec_loop ~50 Hz)
    │
    ▼
M_frame_notification.nas (FrameNotification)
    │
    ├──→ f-14b.nas (F14_exec.update)
    │        ├── timedMotions → spoilers, sweep, nozzles
    │        └── subsystem dispatch
    │
    ├──→ awg_9.nas (radar processing)
    │        ├── rcs.nas (detection range)
    │        └── Target tracking → fire-control.nas
    │
    ├──→ SAS.nas (computeSAS — YASim only)
    │        └── afcs.nas (autopilot coordination)
    │
    ├──→ cadc.nas (computeAPC)
    │
    ├──→ flaps_computer.nas (computeFlaps)
    │        └── sweep_computer.nas (interlock)
    │
    ├──→ sweep_computer.nas (computeSweep)
    │        └── spoilers.nas (wing sweep bias)
    │
    ├──→ spoilers.nas (computeSpoilers — YASim compute, JSBSim read-only)
    │
    ├──→ engines.nas (AICS [JSBSim only], nozzles, EGT)
    │        └── electrics.nas (power state)
    │
    ├──→ gear.nas (NWS, launchbar)
    │        └── M_nearest_carrier.nas
    │
    └──→ instruments.nas (TACAN, ILS, carrier approach)

Weapons Chain:
    weapons.nas (definitions) → pylons.nas (stations)
        → station-manager.nas (framework)
        → fire-control.nas (firing logic)
        → fox2.nas (guidance)

Emesary MP Chain:
    emesary.nas (core)
        → emesary_mp_bridge.nas (encoding)
        → GeoBridgedTransmitter.nas (bridge setup)
        → ArmamentNotification.nas (weapon events)
        → AircraftEventNotification.nas (property sync)

Communications:
    datalink.nas ←→ iff.nas
    an-arc-159v1.nas (UHF comm[1])
    an-arc-182v.nas (VHF/UHF comm[0], nav[0])
    gci-listener.nas (GCI tactical data)
```
