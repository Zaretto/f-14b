# F-14 Tomcat Release Notes - v1.12

**Release Date:** 2026-02-18
**Previous Version:** v1.11 (2019-12-19)

---

## Summary

Release 1.12 is a major update spanning over five years of development.

### Improvements

The flight control system has been substantially reworked based on NASA TM-81833, with new implementations of the pitch, roll, and yaw SAS channels using simulated angular velocity sensors, and a complete rewrite of the Approach Power Compensator.

The aerodynamic model has received extensive updates sourced from NASA TM-X-62306 including new longitudinal data, reworked spoiler and DLC coefficients, revised flaps/slats lift and drag, and corrected pitching moment tables. 

Engine modelling has been improved with revised F-110-GE-400 thrust data and new fuel/oil pressure and temperature instruments for both the F-110-GE-400 and TF-30 variants. 

Carrier operations benefit from an improved arrestor wire simulation and revised glideslope parameters. The cockpit has seen multiple rounds of 3D model improvements including reworked windshield threat indicator lights, better VDI brightness controls, new HSD modes, and corrected instrument positions. Exterior lighting has been completely reworked for both normal and multiplayer visibility. 

The missile guidance code has received numerous updates improving countermeasure resistance, terminal phase update rates, AIM-54 loft profile, and thread safety. Landing gear geometry has been adjusted with fixes to the main strut hard step and kneel system. 

All textures have been converted to power-of-two dimensions for improved GPU compatibility, and the local blackout system has been replaced with the default FlightGear implementation. 

Over 35 bug fixes address issues across flight controls, weapons, radar, cockpit instruments, and multiplayer.

### New Systems

The weapons and damage model has been completely rewritten using the OPRF approved systems bridged over Emesary multiplayer communication, providing visible missile tracking, hit detection, cannon impacts, countermeasure models, and persistent craters across multiplayer sessions. 

New station management system provides structured pylon and store management with selective jettison. 

New RIO cockpit panels have been added for DDI, datalink, IFF, and AAI. The IFF transponder and tactical datalink systems enable identification and target sharing between cooperative aircraft. 

New HUD symbology includes a dynamic launch zone indicator showing missile engagement envelopes and a continuously computed impact point (CCIP) pipper for MK-83 bombs. A combat event log records all engagements with timestamps and callsigns. 

Smoke generators now support independent left/right colour selection.

The AFC MOD 735 DLC is defined by airframe in the livery.

Two new liveries have been added: "Thief of Baghdad" and VF-1 WolfPack. Comprehensive technical documentation has been added covering aerodynamics, FCS, APC, CADC, catapult, electrical, hydraulic, landing gear, radar, and weapons systems.

---

## New Features

### OPRF Multiplayer Weapons and Damage System

The weapons and damage model has been completely rewritten to use the [OPRF approved systems](https://github.com/NikolaiVChr/OpRedFlag) which uses the Emesary publish-subscribe notification system bridged over multiplayer communication, replacing the legacy MP message approach.

- Missile in-flight tracking, hit detection, and damage notifications transmitted via Emesary bridge
- Cannon hits, bomb impacts, and crater effects visible over multiplayer
- Flare and chaff countermeasure models visible to other players
- Event log system records combat engagements with timestamps and callsigns
- Missile Approach Warning (MAW) and Missile Launch Warning (MLW) alerts with RWR integration
- Persistent craters visible over multiplayer when damage is enabled

### Station Management System

A new station management system provides structured pylon and store management.

- New station-manager replaces the previous station class
- Support for selective jettison and per-station store selection
- Automatic shared file updates for damage, missile code, RCS, and vector libraries
- Proper handling of wing fuel tank pylons tied into the fuel system

### RIO Cockpit Panels

New Radar Intercept Officer panels added for the rear cockpit.

- DDI (Digital Data Indicator) panel
- Datalink control panel
- IFF transponder panel with channel selection
- AAI (Angle of Attack Indicator) panel

### IFF and Datalink

Identification Friend or Foe and tactical datalink systems added.

- IFF channel selection from the cockpit
- Datalink target sharing between cooperative aircraft
- Better determination of datalink buddy capability
- Contacts received from buddies beyond own radar range now displayed

### Dynamic Launch Zone Indicator

A new HUD symbology element shows missile engagement envelopes in air-to-air mode.

- Three zones displayed: no-escape (bottom), optimistic (middle), no-hit-chance (top)
- Works with all air-to-air missile types

### Continuously Computed Impact Point (CCIP)

HUD bombing pipper for MK-83 unguided bombs using real-time ballistic computation.

- Pipper crosses out when insufficient time remains for bomb arming
- Uses hud-math module for accurate pipper placement

### Smoke Colour Selection

Left and right smoke generators can now be independently colour-selected for airshow displays.

### AFC MOD 735 (DLC) Livery Support

Livery configuration now supports the AFC MOD 735 Direct Lift Control variant, selectable per livery.

### Combat Event Log

An in-game event log records all combat events (missile launches, hits, kills) with timestamps and callsigns. Accessible from the menu and written to file on exit.

### New Liveries

- "Thief of Baghdad" livery by VooDoo3
- WolfPack (VF-1) livery

### Comprehensive Technical Documentation

Full technical documentation added covering all major aircraft systems, written from the simulation source code and referenced NASA/AFWAL technical reports.

- Aerodynamics, coefficient buildup, and wind tunnel data references
- Approach Power Compensator (APC) and Central Air Data Computer (CADC)
- Catapult launch, electrical, and hydraulic systems
- Landing gear, radar (AWG-9), and weapons systems
- Emesary messaging architecture documentation
- All documentation located in the `documentation/` directory

---

## Improvements

### Flight Control System Rewrite

The FCS has been substantially reworked based on NASA TM-81833 flight control system schematics.

- Analogue FCS implementation from NASA TM-81833
- Yaw SAS (Stability Augmentation System) implemented from TM-81833 reference data
- Roll SAS implemented and flight-tested
- Pitch SAS converted to work in degrees with simulated angular velocity sensors
- SAS roll, yaw, and pitch channel adjustments after flight testing
- CADC Lateral System Authority added
- Keyboard bindings added for individual SAS channel on/off control
- Fix p feedback into yaw channel

### Aerodynamic Model Improvements

Extensive aerodynamic data updates from NASA technical memoranda.

- New longitudinal data from NASA TM-X-62306
- Reworked spoiler and DLC aerodynamic data from TM-X-62306
- Revised flaps and slats coefficients
- Recalculated flaps lift/drag values
- Adjusted pitching moment due to flaps
- Fixed corrupted spoiler pitching moment table
- Added ITS (Integrated Trim System) for pitch with flaps/slats
- Tidied up and reformatted aerodynamic data tables
- Updated F-14A aerodynamic model from F-14B changes
- Landing gear aerodynamic data updated

### APC (Approach Power Compensator) Rewrite

Complete rewrite of the Approach Power Compensator for improved carrier approach handling.

### Carrier Operations

Improved carrier approach and recovery systems.

- Improved arrestor wire simulation
- Carrier approach handling fixes
- Adjusted glideslope parameters
- Changed tuned carrier to nearest carrier logic

### Weapons System Improvements

- AIM-54 now supports maddog (no-lock) firing (patch by bobdotcom)
- AIM-54 dives on target sooner to avoid overshooting
- AIM-54 self-destruct time increased
- AIM-54 minimum range increased with pitbull point 3nm earlier
- AIM-54 midcourse guidance changed to sample-guided until terminal phase
- AIM-9 now uses proportional navigation (PN) guidance law
- AIM-9 no longer reacquires after losing lock (realistic seeker beam width)
- AIM-9 bore loop fix allowing fire without radar lock
- AIM-7 now fully semi-active radar guided
- Semi-active illumination support in Emesary missiles
- Semi-radar guided missiles can resume guiding after chaff decoy
- Support for 3-stage missiles and command-to-line-of-sight (CLOS) guidance
- Warhead type ID 100+ no longer incorrectly classified as cannon shells
- Gun hit reporting scaled to account for reduced submodel fire rate

### Missile Code Updates

Multiple rounds of missile guidance and damage code updates.

- Better countermeasure resistance modelling
- Faster missile update rate during terminal phase
- AIM-54 breaks off climb earlier based on target distance
- Better final in-flight message handling for missiles
- Fix for missiles fired below minimum guiding speed
- Thread safety improvements for cross-thread timer calls

### Radar and RWR

- E-3 AWACS added to RWR threat library
- E-3 Automat added to RCS database
- SA-5 and SA-6 added to RCS, damage, and RWR threat sorting
- A-50 and SU-34 added to RCS table
- RWR now has two threat levels again
- Pilot RWR lines reduced floating over display
- Fix radar database path on macOS and Linux
- Better check to prevent aircraft being designated as ships at low-elevation airports

### Engine Improvements

- F-110-GE-400 revised thrust data
- F-110-GE-400 fuel and oil pressure/temperature instruments
- TF-30 (F-14A) fuel and oil temperature/pressure instruments added
- Engine Z position tuned
- Minor engine fixes

### Cockpit Model Improvements

- 3D cockpit model fixes and improvements across multiple iterations
- Reworked windshield lights (SAM, AI, AAA threat indicators)
- VDI brightness controls improved
- HSD range display fixed with new controls
- Ejection handles and nav display digit fixed
- SEAM LOCK and TRIG HOT pilot cockpit lights animated
- MAW canopy light uses both MAW properties
- Red flood and instrument lighting fixes
- Fuel quantity switch inversion corrected
- Fuel probe cockpit switch fixed
- G meter and displays panel fixed

### Exterior Lighting

- Complete rework of exterior lights for both normal and multiplayer visibility
- Added emissive lighting
- Added missing external lighting models ([#147](https://github.com/Zaretto/fg-aircraft/issues/147))

### Landing Gear

- Gear geometry adjustments ([#171](https://github.com/Zaretto/fg-aircraft/issues/171))
- Fixed hard step on main struts
- Adjusted kneel system and removed inoperative kneel section
- Ground handling adjustments

### Textures

All textures changed to power-of-two dimensions for improved GPU compatibility.

### Blackout System

Removed local blackout system in favour of the default FlightGear blackout system with new parameters.

### Alpha Measurement

New simulation of the angle-of-attack measurement devices.

### Flaps Overspeed

Improved flaps overspeed damage modelling.

### Stall Warning

Fixed stall warning system.

### Sounds

- Sounds moved to avionics bus
- Conditional properties added to sounds for FlightGear 2020.4 compatibility
- Afterburner sound fixes
- RIO view change sound ([#152](https://github.com/Zaretto/fg-aircraft/issues/152))
- Sidewinder growl sound changed (by pinto)
- Radalt warning sound fix
- Spike sound from RWR added
- Differentiated launch and approach warning sounds

### Compatibility

- FlightGear 2017.3 compatibility fixes
- FlightGear 2018.3 compatibility fixes
- FlightGear 2020.4 compatibility (sound conditionals)
- Walk views disabled to avoid conflicts
- Nasal fallback module updates

### README Rewrite

The README has been rewritten as a comprehensive technical overview of the aircraft add-on.

- Documents all NASA/AFWAL/NATOPS references used in the simulation
- Credits all contributors: Alexis Bory (original author), Richard Harrison (primary author), Nikolai V. Chr (major contributor)
- Repository and issue tracker URLs added

---

## Bug Fixes

- [**#209**](https://github.com/Zaretto/fg-aircraft/issues/209): Control stick 3D model position restored to correct cockpit location
- [**#202**](https://github.com/Zaretto/fg-aircraft/issues/202): Added expected aircraft directory
- [**#175**](https://github.com/Zaretto/fg-aircraft/issues/175): Gun hit reporting scaled correctly (each hit reported as 5 due to reduced submodel fire rate)
- [**#171**](https://github.com/Zaretto/fg-aircraft/issues/171): Landing gear geometry adjusted (two rounds of fixes)
- [**#165**](https://github.com/Zaretto/fg-aircraft/issues/165): Added SA-5 and SA-6 to RCS and damage
- [**#164**](https://github.com/Zaretto/fg-aircraft/issues/164): Fix that some callsigns could interfere with Emesary communications
- [**#163**](https://github.com/Zaretto/fg-aircraft/issues/163): Fix radar bug
- [**#159**](https://github.com/Zaretto/fg-aircraft/issues/159): MP RIO fix
- [**#157**](https://github.com/Zaretto/fg-aircraft/issues/157): Radar upgrade
- [**#152**](https://github.com/Zaretto/fg-aircraft/issues/152): RIO view change sound added
- [**#150**](https://github.com/Zaretto/fg-aircraft/issues/150): Replaced local blackout system with default FlightGear blackout
- [**#149**](https://github.com/Zaretto/fg-aircraft/issues/149): External stores select animation reworked
- [**#147**](https://github.com/Zaretto/fg-aircraft/issues/147): Missing external lighting models added
- [**#141**](https://github.com/Zaretto/fg-aircraft/issues/141): Datalink integration
- [**#139**](https://github.com/Zaretto/fg-aircraft/issues/139): IFF channel selection added
- [**#137**](https://github.com/Zaretto/fg-aircraft/issues/137): Revised tyre smoke/spray system
- [**#132**](https://github.com/Zaretto/fg-aircraft/issues/132): FCS improvements (multiple commits)
- [**#121**](https://github.com/Zaretto/fg-aircraft/issues/121): Altitude hold now requires master autopilot to be engaged first
- Wing sweep computer fix (YASim FDM)
- Wing sweep / flaps interaction fix
- Wing sweep rate fixes (variable rate implementation)
- Wing bend model fixes
- ECM scale animation fix
- TACAN fix
- Countermeasure release changed from toggle to momentary assign
- Nasal error fix in payload handling causing inoperable stores
- Nasal error fix in troll logic
- Fix for bomb drops on nil terrain
- Fix JSBSim engine location/orientation warnings
- Autopilot dialog path corrected
- Bad filepath fixes in system definitions
- Fix incorrect low fuel warning
- Removed debug output from nearest carrier logic
- Fix in-air smoke/spray effects
- Fix cockpit fuel probe switch
- Wingtip trail rotation fixes
- Initial lighting switch state fix
- Radalt needle fix
- Initial state fixup for fresh installations
- AOA indexer logic fixes
- Fix that master arm alone would not enable gun firing
- F-14 branding rework

---

## Contributors

- **Richard Harrison** — Primary author: FCS, aerodynamics, APC, cockpit, engines, carrier ops, documentation
- **Nikolai V. Chr** — Weapons, missile code, damage system, Emesary integration, radar, RWR, station management
- **SammySkycrafts** — AIM-9 bore loop fix, damage updates, MP RIO fix
- **Megaf** — IFF, datalink, combat log window, sound conditionals
- **Chris Ringeval** — Sound conditional properties for FlightGear 2020.4
- **Serafeim Liakos (VooDoo3)** — "Thief of Baghdad" livery
- **Thorsten Renk** — Bomb crater effects

---

## Known Issues

None identified at this time.
