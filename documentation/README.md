# F-14 Tomcat Technical Documentation

**Primary Author**: Richard Harrison (rjh@zaretto.com)
**Project**: FlightGear F-14 Tomcat High-Fidelity Simulation
**Website**: http://zaretto.com/f-14
**Last Updated**: 2026-01-27

## Overview

This directory contains comprehensive technical documentation for the F-14 Tomcat simulation, covering the JSBSim flight dynamics model and all major aircraft systems. The documentation is intended for:

- Developers working on the F-14 model
- Flight simulation enthusiasts seeking technical depth
- Virtual pilots learning F-14 systems
- Anyone interested in F-14 aerodynamics and systems modeling

## Documentation Organization

### Aerodynamics and Flight Dynamics

#### [AERODYNAMICS.md](AERODYNAMICS.md)
Complete aerodynamic model documentation covering:
- **Data Sources**: NASA and AFWAL wind tunnel data, references
- **Aircraft Geometry**: Reference dimensions, mass properties
- **Drag Model**: Base drag, induced, compressibility effects
- **Lift Characteristics**: Basic lift, high-lift devices, wing sweep effects
- **Moment Coefficients**: Pitch, roll, yaw characteristics
- **Control Surfaces**: Effectiveness across flight envelope
- **High-Alpha Behavior**: Post-stall characteristics, departure resistance
- **Ground Effects**: Landing dynamics, gear modeling

**Primary References**:
- AFWAL-TR-80-3141 (high-alpha wind tunnel data)
- NASA TM-X-62306 (lateral-directional characteristics)
- NASA TN D-6909 (dynamic stability derivatives)

#### [COEFFICIENT_BUILDUP.md](COEFFICIENT_BUILDUP.md)
Comprehensive technical reference for JSBSim coefficient system:
- **All 6 Aerodynamic Coefficients**: CL (Lift), CD (Drag), CY (Side Force), Cl (Roll), Cm (Pitch), Cn (Yaw)
- **Complete Buildup Formulas**: Every incremental contribution documented
- **Base Coefficients**: ClBAS, CdBAS, CyBAS, etc. with full data tables
- **Control Surface Effects**: Flaps, slats, spoilers, stabilizers with effectiveness tables
- **Special Modifiers**: Cmach factor, ClSweepFactor, wing sweep effects
- **Source Documentation**: Every coefficient linked to NASA/AFWAL reference
- **Sample Data Tables**: Key coefficient values across flight envelope

**Use Cases**:
- Understanding JSBSim aerodynamic model structure
- Modifying or tuning specific coefficients
- Validating model against flight test data
- Developing similar aircraft models

### Major Aircraft Systems

#### [HYDRAULICS.md](HYDRAULICS.md)
Dual hydraulic system with bidirectional transfer:
- **System Architecture**: Flight and Combined systems
- **Bidirectional Transfer**: Cross-system support capability
- **Emergency System**: Electric pump backup
- **Pump Characteristics**: Engine-driven, windmill operation
- **Failure Modes**: Single and multiple pump failures
- **Operating Procedures**: Normal and emergency

**Reference**: NAVAIR 01-F14AAD-1, Pages 2-68 to 2-76

#### [CADC.md](CADC.md) - Central Air Data Computer
Automatic maneuvering flaps/slats and damage modeling:
- **Maneuvering System**: Automatic flap/slat deployment
- **Operating Envelope**: Mach and altitude limits
- **Deployment Logic**: Alpha-based scheduling
- **G-Load Monitoring**: Structural stress tracking
- **Damage Accumulation**: Wing bending, flap overspeed
- **Flutter Modeling**: High-speed aeroelastic effects
- **AOA Indexer**: Approach speed guidance system

**References**:
- NAVAIR 01-F14AAD-1, Figures 2-51, 2-52
- NASA TT F-15,406 (W. Staudacher, 1972)

#### [APC.md](APC.md) - Approach Power Compensator
Automatic throttle control for carrier approaches:
- **Control Law Architecture**: Three-path control system (V-1, V-2, V-3)
- **AOA Maintenance**: Automatic speed control
- **Integration with DLC**: Direct Lift Control compensation
- **Pitch Rate Compensation**: Predictive throttle response
- **Operating Procedures**: Carrier and field approaches
- **Emergency Procedures**: Malfunction handling

**Reference**: NASA TM-X-81833, Figure 38 (Control Law Schematic)

#### [CATAPULT.md](CATAPULT.md)
Carrier catapult launch system:
- **Launch Sequence**: Complete carrier launch procedures
- **Force Modeling**: 3G acceleration profile
- **Kinematic Timing**: 2.7-second stroke duration
- **Safety Interlocks**: Gear down requirements
- **Integration**: Nose gear kneeling, launch bar
- **Emergency Procedures**: Cold/hot cat, engine failures

#### [LANDING_GEAR.md](LANDING_GEAR.md)
Tricycle gear with unique nose kneeling system:
- **Kneeling Function**: Three positions (Kneel, Normal, High)
- **Catapult Operations**: Nose extension for shuttle engagement
- **Deck Storage**: Compressed position for reduced profile
- **Hydraulic Operation**: Extension/retraction system
- **Emergency Procedures**: Emergency extension, gear failures
- **Speed and Weight Limits**: Operational restrictions

#### [ELECTRICAL.md](ELECTRICAL.md)
Simplified 28V DC electrical system:
- **Power Sources**: Battery, generator, external power
- **Distribution**: Master bus architecture
- **Essential Loads**: Avionics, instruments, lighting
- **Circuit Protection**: Breakers and switches
- **Emergency Procedures**: Generator failure, total electrical loss
- **System Interactions**: Hydraulics, CADC, instruments

## Model Fidelity Philosophy

### High-Fidelity Areas

The F-14 model emphasizes accuracy in:

1. **Aerodynamics**: Based on actual NASA/AFWAL wind tunnel data
2. **Flight Control System**: Detailed SAS and AFCS implementation
3. **Variable Geometry**: Accurate wing sweep effects
4. **Carrier Operations**: Catapult, arrestment, approach aids
5. **Weapons Systems**: AWG-9 radar, AIM-54/7/9 missiles
6. **Damage Modeling**: Structural stress, G-limits, flutter

### Simplified Areas

Appropriate simplifications for real-time simulation:

1. **Hydraulics**: Detailed enough for failure training, simplified actuator dynamics
2. **Electrical**: Single bus instead of dual AC/DC systems
3. **Engines**: Accurate thrust/fuel flow, simplified internal dynamics
4. **Avionics**: Functional representation, not cycle-accurate
5. **Fuel System**: Essential plumbing modeled, detailed transfer simplified

### Design Goals

- **Flight Training**: Accurate handling qualities and procedures
- **Emergency Procedures**: Realistic system failures and consequences
- **Carrier Operations**: Full carrier landing/launch capability
- **Combat Systems**: Functional radar and weapons employment
- **Real-Time Performance**: Maintains 60+ FPS on modern hardware

## Key Data Sources

### Primary NASA Documents

1. **AFWAL-TR-80-3141** (1980) - High-alpha wind tunnel data
2. **NASA TM-X-62306** (1973) - Lateral-directional characteristics
3. **NASA TM-X-62244** (1973) - Longitudinal characteristics
4. **NASA TN D-6909** (1972) - Dynamic stability derivatives
5. **NASA-TM-81833** (1980) - Flight control system, APC
6. **NASA-TM-104326** (1996) - Engine performance data

### Flight Manuals

1. **NAVAIR 01-F14AAD-1**: Aircraft Descriptive Data
2. **NAVAIR 01-F14AAP-1**: Flight Manual (NATOPS)
3. Various NASA technical memoranda and reports

### Computational Methods

- **OpenVSP**: Wing sweep and spoiler CFD (where wind tunnel data unavailable)
- **Custom Tools**: Data extraction and table generation

## Development History

### Project Timeline

```
2014-08-02:  Initial F-14A FDM created
2020-12-15:  APC system implemented (NASA TM-X-81833)
2020-12-15:  DLC split spoiler mod (AFC 735)
2026-01-27:  Comprehensive documentation created
```

### Major Milestones

- **Aerodynamic Model**: Years of refinement using multiple NASA sources
- **Flight Control System**: SAS (analog and digital) plus AFCS
- **Weapons System**: Full AWG-9 radar and missile guidance
- **Carrier Operations**: Complete catapult and arrestment systems
- **Damage Model**: Structural stress, flutter, G-limits
- **Dual Control**: Multiplayer-capable pilot/RIO stations

## Using This Documentation

### For Developers

Each document includes:
- **Physical System Description**: How the real aircraft works
- **Implementation Details**: XML code structure and logic
- **Key Properties**: Property tree references
- **System Interactions**: Dependencies between systems

Look for:
- File locations (e.g., `Systems/f-14b-hydraulic.xml`)
- Property paths (e.g., `systems/hydraulics/flight-system-psi`)
- Code snippets showing actual implementation

### For Virtual Pilots

Each document includes:
- **Overview**: What the system does
- **Operating Procedures**: How to use the system
- **Emergency Procedures**: What to do when it fails
- **Limitations**: What's different from the real aircraft

Focus on:
- Operating procedures sections
- Emergency procedures sections
- System interactions (understand dependencies)

### For Flight Sim Enthusiasts

Each document provides:
- **Technical Depth**: Real aerodynamic data and equations
- **References**: Sources for further reading
- **Comparison with Reality**: What's accurate, what's simplified
- **Design Philosophy**: Why certain choices were made

## Contributing

### Reporting Issues

Found an error or have a suggestion?
- Contact: Richard Harrison (rjh@zaretto.com)
- Website: http://zaretto.com/f-14
- FlightGear Forums: F-14 discussion threads

### Documentation Updates

When updating documentation:
1. Maintain technical accuracy (cite sources)
2. Keep formatting consistent
3. Update "Last Updated" date
4. Cross-reference related documents
5. Include property tree paths where relevant

## Additional Resources

### Online Resources

- **Project Website**: http://zaretto.com/f-14
- **FlightGear Forums**: Active F-14 community
- **GitHub Repository**: Source code and issue tracking

### Recommended Reading

For deeper understanding of F-14 systems:
- "Grumman F-14 Tomcat" by Tony Holmes
- "F-14 Tomcat in Action" by Lou Drendel
- NASA Technical Reports (linked in individual documents)
- NATOPS Flight Manual (publicly available portions)

### Video Resources

- Actual F-14 flight footage (YouTube)
- Documentary: "The F-14 Tomcat: A Navy Legend"
- Carrier landing procedures videos
- NASA flight test videos

## Document Index

### By System Category

**Flight Dynamics**:
- [AERODYNAMICS.md](AERODYNAMICS.md) - Complete aerodynamic model

**Propulsion and Fuel**:
- Documentation planned: ENGINES.md, FUEL_SYSTEM.md

**Flight Controls**:
- Documentation planned: FCS.md (Flight Control System)
- Documentation planned: AFCS.md (Automatic Flight Control System)

**Power Systems**:
- [HYDRAULICS.md](HYDRAULICS.md) - Dual hydraulic system
- [ELECTRICAL.md](ELECTRICAL.md) - 28V DC electrical system

**Avionics and Computers**:
- [CADC.md](CADC.md) - Central Air Data Computer
- [APC.md](APC.md) - Approach Power Compensator
- Documentation planned: AWG9_RADAR.md, WEAPONS.md

**Airframe and Landing**:
- [LANDING_GEAR.md](LANDING_GEAR.md) - Gear and kneeling system
- [CATAPULT.md](CATAPULT.md) - Carrier launch system
- Documentation planned: ARRESTING_GEAR.md

**Environmental**:
- Documentation planned: ECS.md (Environmental Control System)
- Documentation planned: OXYGEN.md

### By Complexity Level

**Beginner (Operational Focus)**:
1. [LANDING_GEAR.md](LANDING_GEAR.md) - Straightforward operations
2. [ELECTRICAL.md](ELECTRICAL.md) - Simple power system
3. [CATAPULT.md](CATAPULT.md) - Launch procedures

**Intermediate (System Understanding)**:
1. [HYDRAULICS.md](HYDRAULICS.md) - Dual system complexity
2. [APC.md](APC.md) - Automatic system behavior
3. [CADC.md](CADC.md) - Automated aircraft management

**Advanced (Technical Depth)**:
1. [AERODYNAMICS.md](AERODYNAMICS.md) - Full aero model details
2. Planned: FCS.md - Flight control laws
3. Planned: AWG9_RADAR.md - Radar modeling

## Credits

### Primary Author

**Richard Harrison** (rjh@zaretto.com)
- JSBSim FDM development
- Systems modeling (hydraulics, CADC, APC, etc.)
- FlightGear integration
- Documentation

### Data Sources

- **NASA**: Wind tunnel data, flight test data
- **Air Force Wright Aeronautical Laboratories (AFWAL)**: High-alpha data
- **US Navy**: NATOPS manuals, descriptive data

### Tools and Technologies

- **JSBSim**: Flight Dynamics Model engine
- **FlightGear**: Flight simulation platform
- **OpenVSP**: Computational aerodynamics
- **Various**: Data extraction and processing tools

### Community

Thanks to the FlightGear community for feedback, testing, and support.

---

## License and Disclaimer

This documentation describes a flight simulation model. It is not approved for flight training, engineering analysis, or any safety-critical application. The F-14 Tomcat is a complex military aircraft; this simulation is a best-effort recreation for entertainment and education.

**FlightGear Open Source Project**
**GNU General Public License v2.0 or later**

---

*For questions, corrections, or contributions, please contact Richard Harrison at rjh@zaretto.com or visit http://zaretto.com/f-14*
