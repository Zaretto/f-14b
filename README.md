# FlightGear F-14 Tomcat

A high-fidelity simulation of the Grumman F-14 Tomcat for FlightGear, featuring wind tunnel based aerodynamic data and flight control systems derived from published NASA and Navy technical reports.

**Repository:** https://github.com/Zaretto/f-14b
**Documentation:** http://zaretto.com/f-14
**Issues:** https://github.com/Zaretto/f-14b/issues
**Quick Reference:** [HELP.md](documentation/HELP.md) | **Weapons Guide:** [WEAPONS.md](documentation/WEAPONS.md)

**Technical Documentation:**
- [Aerodynamics](documentation/AERODYNAMICS.md) - Wind tunnel data sources and aerodynamic model 
- [Aerodynamic Coefficients](documentation/COEFFICIENT_BUILDUP.md) - Detailed aerodynamic coefficient buildup methodology
- [APC](documentation/APC.md) - Approach Power Compensator
- [CADC](documentation/CADC.md) - Central Air Data Computer and maneuvering flaps/slats
- [Catapult](documentation/CATAPULT.md) - Catapult launch system
- [Electrical](documentation/ELECTRICAL.md) - Electrical power system
- [Hydraulics](documentation/HYDRAULICS.md) - Hydraulic system
- [Landing Gear](documentation/LANDING_GEAR.md) - Landing gear and kneel system
- [Radar](documentation/RADAR.md) - AWG-9 radar system
- [Weapons](documentation/WEAPONS_SYSTEM.md) - Missile guidance and fire control
- [Emesary](documentation/EMESARY.md) - Inter-object communication system
- [FlightGear Data Flow](documentation/flightgear-data-flow.md) - Property tree and system integration

**Nasal Scripting:**
- [Radar System](documentation/nasal/RADAR.md) - AWG-9 radar implementation and RCS calculation
- [Dual Control](documentation/nasal/DUAL_CONTROL.md) - Pilot/RIO multiplayer synchronisation
- [Fallback Modules](documentation/nasal/FALLBACK.md) - Compatibility layer for older FlightGear versions

**Project:**
- [Release Notes](documentation/release-notes.md) - V1.12 changelog
- [Project History](documentation/project-history.md) - Full development timeline

## Project History

The F-14 Tomcat for FlightGear was created by Enrique Medina and Alexis Bory (xii) in February 2008. Alexis developed the original 3D model, cockpit instruments, Nasal radar system, HUD, fuel system, sounds, liveries, and dual-control multiplayer support over 460 commits through 2012, using a YASim flight dynamics model. Anders Gidenstam contributed the DualControl framework and Stuart Buchanan added air-to-air refueling support during this period.

After a quiet period in 2013, Richard Harrison took over as primary developer in 2014 with the goal of creating an accurate simulation based on publicly available technical documentation rather than estimation. The YASim FDM was replaced with a JSBSim aerodynamic model built from NASA/AFWAL wind tunnel data (AFWAL-TR-80-3141), and engine thrust tables were derived from NASA TM-104326. The F-14A variant with TF-30 engines and compressor stall modelling was added in 2015. onox contributed canopy effects, flight recorder enhancements, and catapult launch fixes during 2014-2016.

Nikolai V. Chr joined as a major contributor in 2016, bringing missile guidance systems (AIM-9, AIM-7, AIM-54), the damage model, RWR, RCS database, and an anti-cheat system. Joshua Davidson implemented the initial Approach Power Compensator in 2019, and V1.9 was released in June 2019.

From 2020-2021 the aircraft underwent a major overhaul. Richard Harrison rewrote the flight control system based on NASA TM-81833, implementing the pitch, roll, and yaw SAS channels with new aerodynamic data from NASA TM-X-62306. Nikolai V. Chr rebuilt the weapons and damage system using the Emesary publish-subscribe framework for multiplayer communication, added the station management system, CCIP bombing pipper, dynamic launch zones on the HUD, and semi-active illumination for missiles. Megaf contributed IFF channel selection and datalink integration. The V2.0 RC1 was released in January 2021.

Development continued through 2022-2024 with radar upgrades, further weapons refinements, new liveries ("Thief of Baghdad" by VooDoo3, VF-1 WolfPack), compatibility fixes, and shared file auto-update integration with the [OPRF](https://github.com/NikolaiVChr/OpRedFlag) systems. Release 1.12 was published in February 2026 with comprehensive technical documentation covering all major aircraft systems.

## Release Notes

### Current Release: V1.12 (February 2026)

**Highlights:**

- Completely rewritten Flight Control System (FCS) based on NASA TM-81833
- Major aerodynamic model updates with new data from NASA TM-X-62306
- Emesary-based weapons and damage system for improved multiplayer combat
- AWG-9 radar upgrades with datalink and IFF integration
- New HUD features including dynamic launch zones and CCIP for bombs
- RIO station improvements with functional DDI, datalink, and IFF panels
- F-110-GE-400 engine model improvements (thrust, oil simulation)
- DLC implementation (original and AFC MOD 735)
- Improved damage model for flaps, slats, wing bending
- New liveries: "Thief of Baghdad", VF-1 WolfPack
- Comprehensive technical documentation for all major systems

**Compatibility:** FlightGear 2017.3+

For the complete changelog, see [Release Notes](documentation/release-notes.md).

---

## Contributors

**Original Author:** Alexis Bory (xii) - Created the original F-14B model including 3D cockpit, instruments, radar, HUD, weapons systems, fuel system, and YASim FDM (2008-2012)

**Primary Author:** Richard Harrison (rjh@zaretto.com) - JSBSim FDM based on NASA/AFWAL wind tunnel data, flight control system, engine models, and ongoing development (2014-present)

**Major Contributor:**
- Nikolai V. Chr - Missile guidance systems (AIM-9/AIM-7/AIM-54), damage model, Emesary multiplayer integration, radar systems, RWR, datalink, IFF, HUD dynamic launch zones, CCIP, RCS database, station manager, fire control, anti-cheat system (337 commits, 2016-present)

**Contributors:**
- Joshua Davidson - Version checking, route manager advance code
- onox - Air-to-air refueling, canopy effects, flight recorder enhancements
- Megaf - Datalink integration, IFF channel selection, combat log window
- SammySkycrafts - AIM-9 bore mode fix, MP RIO fixes, damage system updates
- Thorsten Renk - Bomb crater visual effects
- Stuart Buchanan - Air-to-air refueling support
- Chris Ringeval - Sound system compatibility fixes
- Jmav16 - RCS database contributions
- Paccalin - Missile code improvements
- VooDoo3 - "Thief of Baghdad" livery
- Spectre - Top Gun livery
- Anders Gidenstam - DualControl framework
- Frederic Bouvier, Curtis L. Olson, mfranz, ehofman - FlightGear core compatibility

## Variants

- **F-14A** - Pratt & Whitney TF30-P-414 engines; susceptible to compressor stall at high alpha/sideslip; underpowered for the airframe
- **F-14B** - General Electric F110-GE-400 engines; improved power and reliability

---

## Installation

1. Download or clone the repository
2. Place the `f-14b` folder in your FlightGear `Aircraft` directory
3. Launch FlightGear and select "F-14B" or "F-14A" from the aircraft menu

**Requirements:** FlightGear 2017.3 or later

---

## Quick Start

### Engine Start
1. Pull the fuel cutoff valves (yellow striped handles on glareshield)
2. Select CRANK (with or without external air)
3. Once N2 reaches 18%, hydraulics power the backup electrical generator
4. Cockpit instruments will illuminate

### Flying Notes
- The realistic aerodynamic model makes the aircraft harder to fly than simplified models
- Watch AOA in high-G turns to avoid stalling wings or control surfaces
- Hydraulics and electrics require engine power or external power
- Loss of engines at low speed means loss of flight controls

---

## Keyboard Shortcuts

### Flight Controls
| Key | Function |
|-----|----------|
| Home / End | Elevator trim increase/decrease |
| f / F (Shift-f) | Flaps down/up |
| < / > | Wing sweep forward/aft |
| = | Wing sweep auto mode |
| o | Toggle arrester hook |

### Autopilot & SAS
| Key | Function |
|-----|----------|
| Ctrl-t | Toggle AFCS Attitude Mode |
| Ctrl-a | Enable AFCS Altitude Mode |
| * | Engage AFCS Altitude Mode |
| Ctrl-h | Enable AFCS Heading Mode |
| Ctrl-p | Toggle Pitch SAS |
| Ctrl-k | Toggle Roll SAS |
| Ctrl-l | Toggle Yaw SAS |

### Landing
| Key | Function |
|-----|----------|
| Ctrl-S | Toggle APC (Approach Power Compensator) |
| Ctrl-d | Stick DLC/Chaff switch |
| l / n | DLC thumbwheel fore/aft |
| Ctrl-y | Toggle ground spoilers armed |

### Radar
| Key | Function |
|-----|----------|
| Shift-E / Shift-R | Radar range decrease/increase |
| q | Toggle Radar Standby Mode |
| Ctrl-n | Toggle Radar Mode (Pulse/TWS) |
| h | Cycle HSD modes (radar/compass/ECM) |
| y | Next target |
| Shift-y | Radar azimuth coverage |
| i / I (Shift-i) | Radar up/down |
| D (Shift-d) | Radar elevation coverage |
| r | STT Lock |
| U (Shift-u) | Undesignate |
| d | Pilot automatic lockon mode |

### Weapons
| Key | Function |
|-----|----------|
| Ctrl-w | Cycle Master Arm modes |
| m | Cycle Stick Weapon Selector |
| Shift-m | Cycle pylons selection |
| e | Fire M61A1 Vulcan or selected weapon |

### Countermeasures & Defensive
| Key | Function |
|-----|----------|
| Ctrl-q | Release chaff/flare |
| Ctrl-e | Ignore selected MP pilots |

### Views & Miscellaneous
| Key | Function |
|-----|----------|
| Ctrl-v | Toggle Pilot/RIO view |
| Q (Shift-q) | Reset view |
| c | Toggle canopy and ladder |
| u | Toggle refueling probe |
| Ctrl-o | Toggle wing oversweep (ground only) |
| S (Shift-s) | Toggle smoke |
| F6 | Eject |

### GCI (Ground Control Intercept)
| Key | Function |
|-----|----------|
| Ctrl-1 | Picture request |
| Ctrl-2 | Bogey Dope request |
| Ctrl-3 | Cutoff request |

See [HELP.md](HELP.md) for detailed operational procedures.

---

## Liveries

13 paint schemes available:

| Livery | Description |
|--------|-------------|
| VF-1 WolfPack | Pacific Fleet fighter squadron |
| VF-2 Bounty Hunters | Classic Navy squadron |
| VF-24 Rage 212 | Fighting Renegades |
| VF-101 Grim Reapers | Fleet Replacement Squadron |
| VF-102 Diamondbacks | Atlantic Fleet |
| VF-103 (Santa) | Jolly Rogers holiday special |
| Jolly Rogers | VF-84/VF-103 skull and crossbones |
| Checkmates | VF-211 |
| Swordsmen | VF-32 |
| Red Rippers | VF-11 |
| Top Gun | NFWS adversary |
| NSAWC Aggressor | Naval Strike and Air Warfare Center |
| NASA | NASA Dryden research aircraft (N834NA) |

---

## Technical Documentation

This simulation is built on extensive published research. Each major system references specific technical documents for its implementation.

### Aerodynamic Model

**Primary Data Sources:**

| Reference | Description |
|-----------|-------------|
| AFWAL-TR-80-3141 (Parts I & III) | Core wind tunnel data for high-alpha characteristics at 22° sweep |
| NASA TM-X-62244 | Low-speed longitudinal characteristics in high-lift configuration |
| NASA TM-X-62306 | Low-speed lateral-directional characteristics; flaps, slats, gear, spoiler coefficients |
| NASA TN D-6909 | Dynamic stability derivatives at angles of attack -5° to 90° |
| NASA-TP-3253 | Effects of forebody strakes on aerodynamic characteristics |
| NASA CR-1756 | Used for validation of negative alpha estimates (747-100 data) |

**Implementation Notes:**
- Flaps/slats coefficients from NASA TM-X-62306 are divided: 80% flaps / 20% slats
- Main flaps contribute 70%, auxiliary flaps 30%
- Wing sweep and lateral spoiler effects computed using OpenVSP CFD
- All coefficients validated within 2% of published source values

### Flight Control System

**Primary Reference:** NASA TM-81833 - "Simulator Study of F-14A Utilizing an Aileron-Rudder Interconnect"
**Authors:** Kelly, W. W. and Brown, P. W. (NASA Langley Research Center), May 1980

**Additional Reference:** NASA TM-81972 - "Limited Evaluation of F-14A with Aileron-Rudder Interconnect", December 1981

**Implemented Systems:**
- Pitch damper (SAS) - per Figure 3 with q-rate compensation
- Roll damper (SAS) - per Figure 21
- Yaw damper - per Figure 21
- Pitch stabilizer command via DLC thumbwheel - per Figure 5
- Horizontal stabilizer authority - per NATOPS Figure 2-54
- Lateral system authority - per NATOPS Figure 2-58
- Spoiler gearing schedule - per NATOPS Figure 2-60

### Direct Lift Control (DLC)

Two implementations based on NASA TM-81833:

1. **Original DLC** - All spoilers operate between -4° and +17° for both roll control and DLC
2. **AFC MOD 735** - Inner spoilers dedicated to DLC (full range -4° to +50°), outer spoilers for roll control

### Approach Power Compensator (APC)

**Reference:** NASA TM-81833, Figure 38 - "Schematic of Autothrottle Control Law"

The APC implementation follows the control law schematic diagram exactly as published.

### CADC (Maneuvering Flaps/Slats Computer)

**References:**
- NAVAIR 01-F14AAD-1 - Air Vehicle Description
- NASA TT F-15,406 - "Improvement of Manoeuvrability at Low Speed" (Staudacher, 1972)
- NAVAIR 01-F14AAP-1 Figures 2-51, 2-52 - Extension schedules

**Validation:** Approach speeds verified against NAVAIR 01-F14AAP-1 Figure 7-1 (124-136 knots IAS at 15° AOA with DLC neutral, flaps down)

### Engine Models

#### F110-GE-400 (F-14B)

**References:**
- NASA TM-104326 - Engine thrust tables (Tables 4-6)
- NASA TM-86042 - "Minimum Time and Fuel Flight Profiles for an F-15 with HIDEC"
- NASA TP-1069, TP-1228 - F100 engine calibration data

**Features:**
- Five-stage afterburner with high-voltage ignition modeling
- Pressure-based augmentation light-off (PB > 2.14 threshold)
- EGT calculation based on Mach and fuel flow

#### TF30-P-414 (F-14A)

**Additional References:**
- NASA TP-2461 - "Performance and Surge Limits of TF30-P-3 Engine" (Wasserbauer, Neumann, Shaw, May 1985)
- ASD-TR-75-19 - "J79-15/-17 Turbojet Engine Accident Investigation Procedures"

**Compressor Stall Model** (per F-14A NATOPS pg IV-11-4):
- Mid-Compression Bypass (MCB) active below 35% Mach at subsonic speeds
- Stall margin affected by angle-of-attack and sideslip
- P0(alpha, beta) model for inlet pressure distortion
- Stall triggers when P0-CPV < P0-k4 with N1 > 10%, airborne, and alpha > 4°

### NATOPS Manual References

The simulation references multiple figures from NAVAIR 01-F14AAP-1 (NATOPS Flight Manual):

| Figure | System |
|--------|--------|
| 2-51, 2-52 | Maneuvering flaps/slats schedules |
| 2-54 | Longitudinal system authority |
| 2-58 | Lateral system authority |
| 2-60 | Spoiler gearing schedule |
| 7-1 | Approach speed (DLC neutral) |
| 11-8 | Landing approach airspeed |

---

## Systems Architecture

### Nasal Modules

Key systems implemented in Nasal (loaded in dependency order):

| Module | Purpose |
|--------|---------|
| fox2.nas | Missile guidance (AIM-9, AIM-7, AIM-54) |
| awg-9.nas | AWG-9 radar simulation |
| damage.nas | Damage model with warhead database |
| fuel-system.nas | Multi-tank fuel system with crossfeed |
| fire-control.nas | Weapon firing logic |
| station-manager.nas | 8-pylon store management |

### Communication Patterns

- **Emesary** - Publish-subscribe notification system enabling decoupled communication between classes that have no knowledge of each other. Used extensively for weapons, damage, radar contacts, and system state. Notifications can be bridged across multiplayer. See [EMESARY.md](EMESARY.md) for details.
- **Property Tree** - FlightGear's central data store; all state flows through properties
- **Listeners** - Reactive updates via property change callbacks

### Damage Model

When damage is enabled:
- Flaps and slats receive damage from overspeed conditions
- Continued overspeed causes jammed surfaces
- Excessive G-loading causes wing failure
- Arrestor wire simulation considers aircraft mass and speed

### Dual Control (Multiplayer)

- `--aircraft=f-14b` - Pilot aircraft; backseat RIO can join for radar/weapons/radio
- `--aircraft=f14b-bs` - Backseater mode for joining a dual-control F-14B

---

## Systems Not Yet Modelled

- **ACLS** (Automatic Carrier Landing System) - Awaiting sufficient documentation
- **RATS** (Reduced Thrust Arresting System) for GE engines

---

## Known TODOs and Future Work

The following items are marked as TODO in the codebase for future development:

### Flight Control System
- Autopilot emergency disengage paddle should disengage APC (cadc.nas)

### Weapons System (fox2.nas)
- Side wind compensation for rail heading offset
- Flare/chaff handling improvements
- Seeker FOV pattern checking
- High elevation math improvements
- Horizon stabilized uncaged mode

### Radar System (awg-9.nas, radar-system.nas)
- Scan visibility check interval optimization
- Sweep line visual glitch when changing azimuth scan field
- RWR alert display override
- Chaff detection refinement
- Radar code partitioning (avoid running all 18 contacts at once)
- Vectorized field implementation
- Contact scanning method rework

### Damage System (damage.nas)
- Hash lookup for weapon models
- SAM/ship callsign identification

### Awaiting FlightGear Core Fixes
Several `coord.alt()` calls have workarounds pending upstream FlightGear fixes in:
- fox2.nas (missile coordinates)
- radar-system.nas (aircraft/contact positions)

---

## Validation Approach

The aerodynamic model follows a rigorous validation process:
1. Coefficients compared against original data sources (within 2% of published values)
2. Primary validation through flight testing
3. Cross-validation against NAVAIR 01-F14AAP-1 specifications
4. Photographic references used to verify pitch angles during approach

---

## Additional References

- AFWAL-TR-80-3141: Core wind tunnel data for high-alpha characteristics
- NASA TM-X-62306: Flaps, slats, gear, spoiler coefficients
- NASA TM-81833: Flight control system schematics, DLC, APC
- NASA TM-104326: Engine thrust tables
- NASA TP-2461: TF30 compressor stall characteristics

Detailed aerodynamic data sources: http://zaretto.com/content/f-14-aerodynamic-data-sources

---


## License

This aircraft is released for use with FlightGear Flight Simulator.

Copyright (C) 2014-2026 Richard Harrison and contributors.
