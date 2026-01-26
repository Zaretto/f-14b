# FlightGear F-14 Tomcat

A high-fidelity simulation of the Grumman F-14 Tomcat for FlightGear, featuring wind tunnel based aerodynamic data and flight control systems derived from published NASA and Navy technical reports.

**Repository:** https://github.com/Zaretto/f-14b
**Documentation:** http://zaretto.com/f-14
**Issues:** https://github.com/Zaretto/f-14b/issues

## Project History

Development began in August 2014 with the goal of creating an accurate F-14 simulation based on publicly available technical documentation rather than estimation. The aerodynamic model, flight control system, and engine models are all derived from specific NASA technical memoranda, AFWAL wind tunnel reports, and NATOPS flight manuals.

## Contributors

**Original Author:** Alexis Bory (xii) - Created the original F-14B model including 3D cockpit, instruments, radar, HUD, weapons systems, fuel system, and YASim FDM (2008-2012)

**Primary Author:** Richard Harrison (rjh@zaretto.com) - JSBSim FDM based on NASA/AFWAL wind tunnel data, flight control system, engine models, and ongoing development (2014-present)

**Major Contributor:**
- Nikolai V. Chr - Missile guidance systems (AIM-9/AIM-7/AIM-54), damage model, Emesary multiplayer integration, radar systems, RWR, datalink, IFF, HUD dynamic launch zones, CCIP, RCS database, station manager, fire control, anti-cheat system (337 commits, 2016-present)

**Contributors:**
- Joshua Davidson - Approach Power Compensator (APC) implementation
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

- **Emesary** - Event notification for multiplayer-compatible weapons, damage, and system state
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

## Release Notes

For detailed release notes, see [RELEASE_NOTES.md](RELEASE_NOTES.md).

### Current Release: V1.12 (January 2026)

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

**Compatibility:** FlightGear 2017.3+

See [RELEASE_NOTES.md](RELEASE_NOTES.md) for the complete changelog.

---

## License

This aircraft is released for use with FlightGear Flight Simulator.

Copyright (C) 2014-2026 Richard Harrison and contributors.
