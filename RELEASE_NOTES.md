# F-14 Tomcat Release Notes

**Documentation:** http://zaretto.com/f-14

---

# Release 1.12

**Release Date:** January 2026
**FlightGear Compatibility:** 2017.3+

This release represents over 5 years of development since January 2020, with major improvements to flight dynamics, weapons systems, avionics, and multiplayer capabilities.

---

## Highlights

- **Completely rewritten Flight Control System (FCS)** based on NASA TM-81833 documentation
- **Major aerodynamic model updates** with new data from NASA TM-X-62306
- **Emesary-based weapons and damage system** for improved multiplayer combat
- **AWG-9 radar upgrades** with datalink and IFF integration
- **New HUD features** including dynamic launch zones and CCIP for bombs
- **RIO station improvements** with functional DDI, datalink, and IFF panels

---

## Flight Dynamics Model

### Aerodynamics
- New longitudinal aerodynamic data from NASA TM-X-62306
- Updated landing gear aerodynamic effects
- Revised flaps/slats lift and drag calculations
- Improved pitch moment modeling for flaps
- Corrected spoiler pitching moment tables
- Low speed handling and power/drag tuning improvements
- Wing bend physics corrections
- Variable wing sweep rate now properly modeled
- Auxiliary flaps system implementation

### Flight Control System
- Complete analog FCS implementation based on NASA TM-81833
- Pitch SAS reworked with proper degree-based logic
- Roll SAS implementation with angular velocity sensors
- Yaw SAS damper from NASA TM-81833 technical manual
- Added CADC Lateral System Authority
- SAS channel keybindings for individual on/off control
- Fixed FCS clipto constraints and p-feedback into yaw channel

### Approach Power Compensator (APC)
- Complete APC system rewrite
- AFC MOD 735 DLC implementation (inner spoilers for DLC, outer for roll)
- Livery configuration support for DLC variant selection

### Direct Lift Control (DLC)
- Reworked spoiler and DLC system from TM-X-62306 data
- Improved DLC range and authority

---

## Weapons Systems

### Missiles
- **AIM-54 Phoenix:**
  - Sample-guided until terminal phase
  - "Maddog" (no lock) firing capability (thanks to bobdotcom)
  - Improved dive-down behavior to avoid overshooting target
  - Increased G-force capability
  - Extended self-destruct time
  - Inertial midcourse guidance (harder to spoof)

- **AIM-9 Sidewinder:**
  - Proportional Navigation (PN) guidance law
  - Removed unrealistic reacquisition capability
  - Fixed bore mode loop for firing without radar lock

- **AIM-7 Sparrow:**
  - Full semi-active radar homing implementation
  - Support for semi-active illumination in Emesary system
  - Improved chaff resistance (can resume guiding after being fooled)

- **General Missile Improvements:**
  - 3-stage missile support
  - Command-to-Line-Of-Sight (CLOS) guidance mode
  - Dynamic Launch Zone indicator on HUD (no escape / optimistic / no hit zones)

### Radar & Targeting
- Major AWG-9 radar system upgrade
- Radar database fixes for Mac and Linux
- RWR (Radar Warning Receiver) improvements:
  - Two threat level indication restored
  - Added E-3 AWACS, SA-5, SA-6 to threat database
  - Improved display line positioning
- RCS database additions: A-50, SU-34, E-3 Automat
- Fixed target lock functionality over multiplayer

### Guns
- M61A1 hit reporting fix (5 hits per submodel fire for realistic damage)
- Fixed warhead typeID handling for cannon shells

### Bombs
- Continuously Computed Impact Point (CCIP) pipper on HUD for MK-83
- CCIP crossed out when insufficient time to arm

### Stores Management
- Selective jettison system
- External stores selection improvements
- Station manager integration with pylon system
- Reworked external stores select animation

---

## Avionics & Instruments

### HUD
- Dynamic launch zone indicator for A/A missiles
- CCIP for MK-83 bombs
- Fixed DEST CRS and GS inverted display
- Fixed Mach display

### RIO Station
- New DDI, datalink, IFF, and AAI panels
- TID heading indication fix for radar returns
- VDI brightness controls improved
- HSD range display fix with new controls

### Pilot Instruments
- Radar altimeter completely redone based on NATOPS
- Airspeed/Mach indicator improvements
- G meter and displays panel fixes
- Fixed radalt needles
- SEAM LOCK and TRIG HOT cockpit light animations
- New method for checking weapon ready status

### Navigation
- TACAN fixes
- Fixed ejection handles and nav display digit

### IFF System
- IFF channel selection implementation
- Encapsulated IFF in instrumentation system

### Datalink
- Full datalink system implementation
- Improved buddy target info transmission
- Fixed display of contacts beyond radar range

---

## Engines

### F110-GE-400 (F-14B)
- Revised thrust data from NASA TM-104326
- Fuel and oil pressure/temperature simulation
- Fixed JSBSim engine location/orientation warnings

### TF30-P-414 (F-14A)
- Added fuel and oil temperature/pressure gauges
- Fixed channel names
- Compressor stall malfunction (F-14A only)

### General
- Improved oil temperature and pressure modeling
- Fixed low fuel warning
- Master caution system converted to property rules

---

## Cockpit & Model

### 3D Model Improvements
- Extensive cockpit model fixes and improvements
- Texture optimization (power of two dimensions)
- Wing bend corrections
- Fixed stick position (issue #209)

### Lighting
- AOA Indexer rework with correct logic
- Red flood lighting fixes
- Emissive lighting for cockpit lights
- Exterior lights rework (MP compatible)
- Windshield threat warning lights (SAM/AI/AAA)
- Fixed initial state of lighting switches

### Animations
- ECM scale animation fixes
- External stores animation rework

---

## Multiplayer & Emesary

### Damage System
- Complete Emesary-based damage system
- Warhead database updates
- Persistent craters over MP
- Hit smoke visualization over MP
- Anti-cheat system (preventive rather than reactive)
- Cannon/rocket type identification
- SA-5 and SA-6 added to damage model

### Weapons Over MP
- Flare models visible over MP
- Missile flight notifications
- Emesary model transmission
- Event log system

### RIO Dual Control
- MP RIO fixes
- View change sound for RIO

### Communication
- Fixed callsign parsing issues in Emesary
- Radar lock spike notifications to MP aircraft
- Improved bridge messaging

---

## Visual Effects

### Smoke & Trails
- Revised tyre smoke/spray system
- Wingtip contrail rotation fixes
- Fixed smoke trail expressions
- Left/Right smoke color selection

### Afterburner
- Fixed afterburner sounds

### General
- Flare model improvements
- Countermeasure release (fixed toggle vs. assign behavior)

---

## Landing Gear & Carrier Operations

### Gear
- Adjusted gear geometry (multiple fixes)
- Fixed hard step on main struts
- Kneel system adjustments
- Ground handling improvements
- Disabled walk views

### Carrier
- Improved arrestor wire simulation
- Changed to nearest carrier detection (vs tuned carrier)
- Carrier approach fixes

---

## Compatibility & Stability

- FlightGear 2017.3, 2018.3, and 2020.4+ compatibility
- XML/XSLT/XSD validation fixes
- Fallback modules for older FG versions
- Auto-updater for shared files (damage, missile-code, etc.)
- Fixed various Nasal errors
- Removed unused code and cleaned up initialization

---

## Liveries

- Added "Thief of Baghdad" livery (by VooDoo3)
- Added VF-1 WolfPack livery
- Livery support for AFC MOD 735 DLC configuration

---

## Bug Fixes

- Fixed autopilot altitude hold engagement logic (#121)
- Fixed external lighting models (#147)
- Fixed payloads becoming inoperable (#149)
- Fixed MAW canopy warning light (#152)
- Fixed gear geometry (#171)
- Fixed gun hit reporting (#175)
- Fixed expected aircraft directory (#202)
- Fixed control stick position (#209)
- Fixed inverted fuel quantity switch
- Fixed cockpit fuel probe switch
- Fixed fuel restore behavior
- Numerous minor fixes throughout

---

## Documentation

- Updated README
- Reworked branding

---

## Contributors

This release includes contributions from multiple developers:

**Original Author:**
- Alexis Bory (xii) - Created the original F-14B model: 3D cockpit, instruments, radar, HUD, weapons, fuel system, YASim FDM (2008-2012)

**Primary Author:**
- Richard Harrison - JSBSim FDM from NASA/AFWAL data, flight control system, engine models (2014-present)

**Major Contributor:**
- Nikolai V. Chr - Missile systems, damage model, Emesary MP integration, radar, RWR, datalink, IFF, HUD features, RCS database, station manager, fire control (2016-present)

**Contributors:**
- Joshua Davidson (APC implementation)
- onox (AAR, canopy effects, flight recorder)
- Megaf (datalink, IFF, combat log)
- SammySkycrafts (AIM-9 fixes, MP RIO, damage system)
- Thorsten Renk (bomb crater effects)
- Stuart Buchanan (AAR support)
- Chris Ringeval (sound compatibility)
- Jmav16 (RCS data)
- Paccalin (missile code)
- VooDoo3 (Thief of Baghdad livery)
- Spectre (Top Gun livery)
- And all other contributors and testers

---

## Known Issues

See the issue tracker at: https://github.com/Zaretto/f-14b/issues

---

## Upgrade Notes

- Saved fuel/loadout configurations may need to be recreated
- Some keyboard bindings have changed - check f-14-common.xml
- Users of older FlightGear versions should verify fallback module compatibility
