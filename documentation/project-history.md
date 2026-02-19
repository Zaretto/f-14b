# F-14 Tomcat for FlightGear — Project History

**Repository:** https://github.com/Zaretto/fg-aircraft
**Total commits:** 1,451 across 18 contributors (February 2008 – February 2026)

---

## Origins (February – December 2008)

The F-14 Tomcat for FlightGear was created on 19 February 2008 by Enrique Medina, whose first commit described it as "A brand new Tomcat, early alpha, comments are welcome." The initial commits, made via Alexis Bory's repository account, established the YASim flight dynamics model, Nasal-based SAS, and basic textures.

Alexis Bory (xii) rapidly built out the aircraft over the course of 2008 with 232 commits, creating:

- 3D cockpit with working instruments (altimeter, VSI, airspeed indicator, chronograph)
- Nasal radar system with carrier detection and different symbology
- Head-Up Display showing magnetic heading
- Fuel system built with Vivian's Buccaneer tools
- Cockpit and engine sounds
- Multiplayer-visible animations
- Livery system with three initial liveries by Flyingtoaster
- Refuelling probe switch
- Electrical system definition
- AFCS (Automatic Flight Control System) with roll SAS
- Landing gear tuning for carrier deck operations

FlightGear core contributors mfranz (2 commits) and ehofman (1 commit) provided compatibility fixes during this period.

---

## Expansion (2009)

Alexis Bory continued as sole developer with 134 commits, adding:

- Dual-control multiplayer (June 2009) — the first step toward a pilot/RIO two-seat experience
- Refuelling probe cycling via keyboard
- Continued cockpit instrument development
- Radar display improvements
- General refinements and bug fixes

---

## Maturation (2010 – 2012)

Alexis Bory contributed a further 72 commits across these three years (under varying author names: "abory", "Alexis Bory", "Alexis"), bringing the total to approximately 462 commits.

Key developments:

- **2010:** Dual-control enhancements — back-seater ECM and compass displays, basic radar controls for the RIO. Anders Gidenstam contributed the DualControl framework, moving it to the generic Aircraft/Generic/DualControl/ location.
- **2011:** Continued instrument and systems refinements. Curtis L. Olson provided a compatibility fix.
- **2012:** TACAN panel standardisation, fuel system bug fixes, further cockpit work. Frederic Bouvier and ThorstenB contributed FlightGear core compatibility patches.

By the end of 2012, the aircraft had a complete YASim-based flight model, working cockpit, basic radar, dual-control multiplayer, fuel system, liveries, and carrier operations — a solid foundation for the next phase of development.

---

## Quiet Period (2013)

No commits were recorded in 2013. Development was on hiatus.

---

## New Leadership and JSBSim (2014 – 2015)

Richard Harrison took over as primary developer with the goal of creating an accurate F-14 simulation based on publicly available NASA and Navy technical documentation.

**2014:**
- Stuart Buchanan added air-to-air refueling support (January 2014)
- Richard Harrison began replacing the YASim FDM with a JSBSim aerodynamic model
- Engine thrust tables derived from NASA TM-104326
- Fuel system properties reworked for the new FDM
- Wing sweep and control surface properties updated
- Specific Fuel Consumption revised from Navy documentation
- onox contributed external engine sounds through open canopy and flight recorder enhancements (November 2014)

**2015:**
- The YASim version was removed and replaced entirely with JSBSim
- F-14A variant added with Pratt & Whitney TF-30-P-414 engines
- onox fixed wing breakage during catapult launch and added canopy reflection effects
- ALS panel backlighting and instrument illumination
- New altimeter model based on photographic reference
- Instrument reworking and lighting improvements
- Aerodynamic breakpoint table harmonisation

---

## Combat Systems and Radar (2016 – 2019)

Nikolai V. Chr joined as a major contributor in 2016, bringing extensive weapons and combat systems expertise across approximately 336 commits.

**2016:**
- Nikolai V. Chr added anti-cheat system with refueling exception for tanker connection
- Livery updates (Bounty Hunters)
- onox added AAR type (boom/probe) for refueling over multiplayer

**2017:**
- Richard Harrison fixed liveries
- Compatibility work

**2018:**
- Richard Harrison reworked the HUD (new eyepoint fixes, waypoint display, TACAN positioning, pressure VSI definitions)
- Carrier landing tuning — revisions to aerodynamics and APC based on Case I approach testing
- Carrier and escort replay capability added
- Carrier approach initialisation improvements
- AWG-9 radar fixes
- External stores model unification
- Hook overspeed logic

**2019:**
- Joshua Davidson implemented the Approach Power Compensator (APC) with military thrust disable logic (January 2019)
- Richard Harrison fixed radar, AWG-9, and JSBSim system model compatibility
- Livery patch fixes for Pilot/RIO
- J Maverick 16 updated the Top Gun livery (by Spectre)
- Nikolai V. Chr and Jmav16 updated the RCS database
- **V1.9 released** (June 2019)
- Richard Harrison fixed the splash screen and version branding

---

## Major Overhaul (2020 – 2021)

This period saw the most intensive development since the original creation, with fundamental rewrites of the flight control system, weapons, and damage model.

### 2020

**Emesary Multiplayer Weapons (February – November):**

Richard Harrison and Nikolai V. Chr introduced the Emesary publish-subscribe notification system for multiplayer weapons communication:

- Richard Harrison created the initial Emesary-based armament notification system (February 2020)
- Thorsten Renk added bomb crater visual effects
- Nikolai V. Chr progressively built out the full Emesary weapons infrastructure:
  - Missile in-flight notifications transmitted over multiplayer
  - Hit detection, damage notifications, and cannon hit messages
  - Missile Approach Warning (MAW) and Missile Launch Warning (MLW)
  - Flare and chaff countermeasure models visible to other players
  - Persistent craters over multiplayer
  - Event log system for combat engagements
  - Thread safety improvements for cross-thread timer calls

**Flight Control System (March – December):**

Richard Harrison began the FCS rewrite based on NASA TM-81833:

- Yaw SAS implemented from TM-81833 reference data (April 2020)
- Roll SAS implemented and flight-tested (April 2020)
- New aerodynamic data from NASA TM-X-62306 — spoilers, DLC, flaps longitudinal data
- Corrected spoiler pitching moment table
- Recalculated flaps lift/drag values
- APC rewrite (December 2020)
- New landing gear aerodynamic data

**Other 2020 work:**
- Carrier approach handling fixes
- Low-speed handling and power/drag tuning
- Wing sweep/flaps interaction fix
- WolfPack (VF-1) livery added
- Improved arrestor wire simulation

### 2021

**Flight Control System (continued):**

- Pitch SAS converted to work in degrees with simulated angular velocity sensors
- Roll and yaw SAS channel adjustments after flight testing
- CADC Lateral System Authority added
- Keyboard bindings for individual SAS channel on/off control
- New simulation of the angle-of-attack measurement devices
- F-14A aerodynamic model release 1.2 (January 2021)
- **V2.0 RC1** (January 2021)

**Emesary Station Management (May 2021):**

Nikolai V. Chr delivered a major combined update installing:
- Station management system replacing the previous station class
- Emesary damage system
- IFF and datalink
- CCIP pipper in HUD for MK-83 bombs
- Flare model visibility over multiplayer
- Radar lock spike notifications
- RCS updates
- AIM-7 full semi-active guidance
- Anti-cheat system changes

**Other contributors (2021):**
- Megaf added IFF channel selection and datalink integration (May 2021)
- Megaf made the combat log window resizable
- Chris Ringeval added sound conditional properties for FlightGear 2020.4 compatibility

**Other 2021 work by Richard Harrison:**
- F-110-GE-400 revised thrust data
- Fuel and oil pressure/temperature instruments for both engine variants
- Master caution system rewritten as property rules
- Cockpit model improvements (multiple rounds)
- RIO panels added (DDI, datalink, IFF, AAI)
- VDI brightness controls
- HSD range display and new controls
- Reworked windshield threat indicator lights
- AOA indexer logic fixes and lighting rework
- Exterior lighting rework
- Missing external lighting models added
- Radalt redone based on NATOPS
- Mach display fix on airspeed indicator
- Flaps overspeed damage improvements
- Smoke colour selection (left/right independent)
- AFC MOD 735 DLC livery support
- Blackout system replaced with default FlightGear implementation
- Compatibility with FlightGear 2018.3

---

## Weapons and Radar Refinement (2022 – 2023)

### 2022

**Nikolai V. Chr:**
- F-14 Radar upgrade ([#157](https://github.com/Zaretto/fg-aircraft/issues/157)) — major AWG-9 improvements (February)
- F-14 and F-15 weapons update ([#162](https://github.com/Zaretto/fg-aircraft/issues/162)) (July)
- Fix that some callsigns could interfere with Emesary communications ([#164](https://github.com/Zaretto/fg-aircraft/issues/164))
- Added SA-5 and SA-6 to RCS and damage ([#165](https://github.com/Zaretto/fg-aircraft/issues/165))
- RWR improvements — two threat levels, reduced line floating, E-3 AWACS and A-50/SU-34 added
- Datalink improvements — better buddy capability detection, contacts beyond radar range
- Semi-radar guided missiles can resume guiding after chaff decoy
- Support for 3-stage missiles and command-to-line-of-sight (CLOS) guidance
- AIM-54 midcourse guidance changed to inertial
- Continuously computed impact point (CCIP) added to HUD for MK-83 (December)
- Dynamic launch zone indicator added to HUD for air-to-air missiles (December)

**Richard Harrison:**
- Selective jettison and stores selection (January)
- Fuel restore rework
- Wingtip trail rotation fixes
- In-air smoke/spray fixes
- JSBSim engine location/orientation warning fixes
- Wing bend model fixes
- Branding rework

**SammySkycrafts:**
- Damage file updates
- MP RIO fix ([#159](https://github.com/Zaretto/fg-aircraft/issues/159))

### 2023

**Nikolai V. Chr:**
- Semi-active illumination support in Emesary missiles
- AIM-54 sample-guided until terminal phase
- AIM-9 proportional navigation guidance law
- AIM-9 realistic seeker beam width (no reacquisition after losing lock)
- ECM warning system re-enabled
- Countermeasure release changed from toggle to momentary assign
- Radar database path fix for macOS and Linux
- Shared file auto-update system installed for damage, missile code, RCS, and vector libraries
- Station-manager updated for new shared file system

**Richard Harrison:**
- Wing sweep rate fixes (variable rate implementation)
- Exterior lighting — emissive lights, power-of-two textures
- Landing gear geometry adjustments ([#171](https://github.com/Zaretto/fg-aircraft/issues/171))
- Main strut hard step fix
- Kneel system adjustments
- Ground handling improvements
- TF-30 fuel and oil temperature/pressure instruments
- Stall warning fix
- Sounds moved to avionics bus
- Nearest carrier logic rework
- Windshield lights renamed (SAM, AI, AAA)
- Compatibility fixes for FlightGear 2017.3 and 2018.3

**Serafeim Liakos (VooDoo3):**
- "Thief of Baghdad" livery (January 2023)

---

## Maintenance and Release (2024 – 2026)

### 2024

- Nikolai V. Chr: AIM-54 maddog firing support (patch by bobdotcom), AIM-54 dive timing and G-force improvements, self-destruct time increase
- Nikolai V. Chr: Warhead type ID 100+ no longer incorrectly classified as cannon shells
- Richard Harrison: AIM-9 bore loop fix allowing fire without radar lock (by SammySkycrafts)
- Richard Harrison: TACAN fix, ECM scale animation fix, version number updates
- Richard Harrison: Engine minor fixes, FlightGear 2017.3 compatibility
- Richard Harrison: Walk views disabled to avoid conflicts

### 2025

- Richard Harrison: G meter and displays panel fix
- Richard Harrison: Control stick position restored ([#209](https://github.com/Zaretto/fg-aircraft/issues/209))

### 2026

- Richard Harrison: Comprehensive release notes and README rewrite with contributor credits and technical references (January 2026)
- Richard Harrison: Wing sweep computer fix for YASim FDM (January 2026)
- Richard Harrison: Comprehensive technical documentation for all major aircraft systems — aerodynamics, FCS, APC, CADC, catapult, electrical, hydraulics, landing gear, radar, weapons (February 2026)
- **Release 1.12** published (February 2026)

---

## Contributors

| Contributor | Commits | Period | Primary Contributions |
|-------------|---------|--------|----------------------|
| Alexis Bory (xii) | ~462 | 2008–2012 | Original aircraft, 3D model, YASim FDM, cockpit instruments, radar, HUD, fuel system, sounds, liveries, dual control |
| Richard Harrison | ~613 | 2014–present | JSBSim FDM (NASA/AFWAL data), FCS, SAS, APC, engines, cockpit, carrier ops, documentation |
| Nikolai V. Chr | ~336 | 2016–present | Missile guidance (AIM-9/7/54), damage model, Emesary MP integration, radar, RWR, RCS, datalink, IFF, HUD DLZ/CCIP, station manager, fire control |
| Joshua Davidson | 8 | 2019 | Approach Power Compensator implementation |
| onox | 7 | 2014–2016 | Canopy effects, flight recorder, catapult fix, AAR over MP |
| Megaf | 6 | 2021 | IFF channel selection, datalink, combat log window |
| SammySkycrafts | 4 | 2022 | AIM-9 bore mode fix, damage updates, MP RIO fix |
| Enrique Medina | — | 2008 | Initial FDM and textures (commits via Alexis Bory's account) |
| Stuart Buchanan | 1 | 2014 | Air-to-air refueling support |
| Anders Gidenstam | 1 | 2010 | DualControl framework |
| Chris Ringeval | 1 | 2023 | Sound conditional properties for FlightGear 2020.4 |
| Thorsten Renk | 1 | 2020 | Bomb crater visual effects |
| Serafeim Liakos (VooDoo3) | 1 | 2023 | "Thief of Baghdad" livery |
| J Maverick 16 | 1 | 2019 | Top Gun livery update |
| Frederic Bouvier | 1 | 2012 | FlightGear core compatibility |
| Curtis L. Olson | 1 | 2011 | FlightGear core compatibility |
| mfranz | 2 | 2008 | FlightGear core compatibility |
| ehofman | 1 | 2008 | FlightGear core compatibility |

---

## Version History

| Version | Date | Milestone |
|---------|------|-----------|
| Alpha | February 2008 | Initial creation by Enrique Medina and Alexis Bory |
| V1.9 | June 2019 | JSBSim FDM, F-14A variant, combat systems |
| F-14A Aero 1.2 | January 2021 | Revised F-14A aerodynamic model |
| V2.0 RC1 | January 2021 | FCS rewrite, Emesary weapons, station management |
| V1.12 | February 2026 | Current release — [full release notes](release-notes.md) |
