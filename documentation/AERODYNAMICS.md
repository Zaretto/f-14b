# F-14 Tomcat Aerodynamics Documentation

**Author**: Richard Harrison (rjh@zaretto.com)
**Last Updated**: 2026-01-27
**Model Version**: 1.2

## Overview

This document describes the aerodynamic model of the F-14 Tomcat as implemented in the JSBSim flight dynamics engine. The model is based on extensive NASA and Air Force wind tunnel data, providing high-fidelity representation of the aircraft's aerodynamic characteristics across a wide range of flight conditions.

## Primary Data Sources

The aerodynamic model draws from multiple authoritative sources:

### Core Wind Tunnel Data

1. **AFWAL-TR-80-3141 Parts I and III** (1980)
   - Authors: Donald E. Johnston, David G. Mitchell, Thomas T. Myers
   - Title: "Investigation of High-Angle-of-Attack Maneuver-limiting factors"
   - Primary source for high-alpha characteristics
   - Data at 22° wing sweep, positive alpha, clean configuration, low speed

2. **NASA TM-X-62306** (October 1973)
   - Authors: William T. Eckert and Ralph L. Maki
   - Title: "Low-speed wind tunnel investigation of the lateral-directional characteristics of a large-scale variable wing-sweep fighter model in the high-lift configuration"
   - Source for flaps, speedbrakes, landing gear, and spoiler coefficients

3. **NASA TM-X-62244** (August 1973)
   - Authors: William T. Eckert and Ralph L. Maki
   - Title: "Low-speed wind tunnel investigation of the longitudinal characteristics"
   - Longitudinal data for high-lift configuration

4. **NASA TN D-6909** (1972)
   - Authors: Sue B. Grafton and Ernie L. Anglin
   - Title: "Dynamic Stability Derivatives at Angles of Attack from -5° to 90°"
   - Dynamic stability characteristics for variable-sweep configuration

5. **NASA-TM-81833** (May 1980)
   - Authors: W. W. Kelly and P. W. Brown
   - Title: "Simulator results of an F-14A airplane utilizing an aileron-rudder interconnect"
   - Flight control system data, DLC, APC schematics

6. **NASA-TM-104326** (November 1996)
   - Authors: Jon K. Holzman, Lannie D. Webb, Frank W. Burcham Jr.
   - Title: "Flight and Static Exhaust Flow Properties of an F110-GE-129 Engine"
   - Engine thrust tables

### Computational Methods

- **Wing Sweep Effects**: Based on CFD computed values using OpenVSP
- **Lateral Spoilers**: CFD computed values (OpenVSP)
- **Mach Number Corrections**: NASA-TM-84643 Figure 3 (Cl-alpha)
- **Transonic Drag**: NASA-aaia-2000-0900 Figure 4

## Aircraft Geometry and Reference Dimensions

### Basic Dimensions

```
Wing Area:           565 ft²
Wingspan (unswept):  64 ft
Wing Incidence:      2°
Mean Aerodynamic Chord: 9.82 ft

Horizontal Tail:
  Area:              140 ft²
  Arm:               25.10 ft

Vertical Tail (twin):
  Total Area:        118 ft² (59 ft² each)
  Arm:               25.10 ft
```

### Reference Points

All measurements from nose:

- **VRP (Visual Reference Point)**: x=386", y=0", z=31"
- **AERORP (Aerodynamic Reference Point)**: x=419", y=0", z=0"
- **CG (Center of Gravity, empty)**: x=407", y=0", z=0"
- **Eyepoint (Pilot)**: x=197", y=0", z=-3.94"

### Mass Properties

```
Empty Weight:  42,104 lbs
Ixx:           58,950 slug·ft²
Iyy:           225,600 slug·ft²
Izz:           285,000 slug·ft²
```

## Coefficient Structure

The JSBSim model uses standard aerodynamic coefficients:

### Force Coefficients

- **CD** (Drag Coefficient): Drag force / (dynamic pressure × wing area)
- **CY** (Side Force Coefficient): Side force / (dynamic pressure × wing area)
- **CL** (Lift Coefficient): Lift force / (dynamic pressure × wing area)

### Moment Coefficients

- **Cl** (Rolling Moment): Roll moment / (dynamic pressure × wing area × span)
- **Cm** (Pitching Moment): Pitch moment / (dynamic pressure × wing area × chord)
- **Cn** (Yawing Moment): Yaw moment / (dynamic pressure × wing area × span)

## Drag Model

### Base Drag Coefficient (CdBAS)

The base drag coefficient (CdBAS) is the reference parasitic drag at a baseline configuration. It varies with angle of attack, Mach number, sideslip, and wing sweep:

```
CdBAS = f(α, M, β, sweep)
```

Characteristics:
- Minimum drag at α ≈ 0° to 5°
- Increases significantly at high alpha (>30°) due to separated flow and increased pressure drag
- Compressibility effects modeled via Mach tables
- Includes skin friction, form drag, and interference drag at reference condition

**Note**: CdBAS is the *base* drag coefficient, not zero-lift drag (CD0). The total drag is built up from CdBAS by adding incremental contributions from configuration changes, control surface deflections, and induced drag effects.

### Mach Factor Correction (Cmach)

Applied to all coefficients except drag:

```
Correction Factor = f(α, M)

At α = 0°, M = 0.9: Factor = 1.240
At α = 0°, M = 2.0: Factor = 0.516
```

This accounts for compressibility effects on aerodynamic forces across the flight envelope from subsonic through supersonic speeds.

### Induced Drag

Implicitly modeled through lift-dependent components and wing sweep effects.

### Incremental Drag Sources

1. **Landing Gear** (CddLandingGear)
   - From NASA TM-X-62306 Test 4
   - Significant drag increase across all alpha ranges
   - Maximum at low alpha angles

2. **Speedbrake** (CddSpeedbrake)
   - Variable deployment 0° to 60°
   - Maximum effectiveness at moderate alpha

3. **External Tanks** (CdTNK)
   - Fitted/not fitted flag
   - Mach-dependent drag increase
   - Wave drag spike near M=0.9

4. **Flaps and Slats**
   - Main flaps: 0° to 35°
   - Auxiliary flaps: 0° to 35°
   - Slats: 0° to 17°
   - Each contributes alpha-dependent drag

5. **Spoilers**
   - Four independent sections (left/right, inner/outer)
   - Used for roll control and Direct Lift Control (DLC)
   - Drag contribution proportional to deflection

6. **Wing Asymmetry Damage** (CdDasym)
   - Models catastrophic wing damage
   - Significant drag increase when active

## Lift Characteristics

### Basic Lift (ClBAS)

The lift coefficient is primarily driven by angle of attack and modified by numerous factors:

```
ClBAS = f(α, β, sweep)
```

Key characteristics from AFWAL-TR-80-3141 Part III Figure 36:

- **CLα (Lift Curve Slope)**: Approximately 0.10 per degree at low alpha
- **CLmax**: Approximately 1.27 at α = 15° (clean, 22° sweep)
- **CLmin**: Approximately -1.03 at α = -10°
- **Stall Characteristics**: Progressive stall beyond 15° alpha

### Wing Sweep Effects (ClSweepFactor)

From NASA-TM-D-6909, sweep dramatically affects lift:

```
Sweep    0°    22°    50°    58°
Factor   1.0   1.0    0.79   0.65  (at α = 0°)

At α = 30°:
Factor   1.0   1.0    1.03   0.96
```

Notable behavior:
- At low alpha, aft sweep reduces lift
- At high alpha (25°-40°), swept configuration can produce slightly more lift
- Complex non-linear relationship

### High-Lift Devices

#### Main Flaps (CldMainFlaps)

From NASA TM-X-62306, flaps contribute 70% of total flap/slat lift increment:

```
Deflection:  0° to 35°
Peak ΔCL:    0.510 at α = 19°, δf = 35°
```

#### Auxiliary Flaps (CldAuxFlaps)

Contribute 30% of total flap increment:

```
Deflection:  0° to 35°
Peak ΔCL:    0.219 at α = 19°, δf = 35°
```

#### Slats (CldSlats)

Leading edge slats contribute 20% of total high-lift increment:

```
Deflection:  0° to 17°
Peak ΔCL:    0.182 at α = 19°, δs = 17°
```

### Spoiler Lift Effects

Each of four spoiler sections reduces lift when deployed:

```
Inner/Outer Spoilers:
  Deflection: -4° to +50° (inner), -4° to +17° (outer)
  ΔCL: Approximately -0.005 per degree at moderate alpha
  Effectiveness reduces above α = 15°
```

### Ground Effect

Modeled through contact points and gear compression. Close to ground, effective angle of attack increases, providing additional lift cushion during landing.

## Pitch Moment (Cm)

### Basic Pitching Moment (CMBAS)

From AFWAL-TR-80-3141:

```
CMBAS = f(α, β)

Characteristics:
- Slightly positive at α = 0° (Cm ≈ +0.072)
- Becomes strongly negative beyond α = 10°
- Minimum Cm ≈ -0.64 at α = 55°
```

The aircraft exhibits strong pitch-down tendency at high alpha, aiding recovery from departed flight.

### Stabilizer/Elevator Control

Two regions of deflection modeled:

1. **Region 1** (-10° ≤ δe ≤ +10°): CMDS1 coefficients
   - ΔCm/Δδe ≈ -0.016 per degree

2. **Region 2** (δe > +10°): CMDS2 coefficients
   - Different effectiveness beyond +10° deflection
   - Captures non-linear behavior at high elevator angles

Total elevator authority:
```
Forward:  -12° (Trailing Edge Down - nose down)
Aft:      +33° (Trailing Edge Up - nose up)
Rate:     36°/second
```

### Pitch Damping (CMQ)

From NASA TN D-6909:

```
CMQ = (c̄q)/(2V₀) × coefficient table

Values:
α = 0°:   CMQ ≈ -15.4
α = 15°:  CMQ ≈ -22.8
α = 35°:  CMQ ≈ -1.9
```

Strong pitch damping at moderate alpha improves handling. Reduced damping at very high alpha reflects reduced stabilizer effectiveness.

### Configuration Effects on Pitch

1. **Speedbrake** (CMMdSpeedbrake)
   - Pitch-up moment when deployed
   - Compensates for drag increase

2. **Landing Gear** (CMMdLandingGear)
   - Pitch-down moment when extended
   - Varies significantly with alpha

3. **Spoilers** (4 sections)
   - Pitch-up moment when deployed
   - Used in DLC (Direct Lift Control) mode

4. **External Stores**
   - AIM missiles create pitch moments (DCMAIM)
   - Mach and alpha dependent

## Roll Moment (Cl)

### Basic Rolling Moment (CLBAS)

Primarily induced by sideslip:

```
CLBAS = f(α, β)

Dihedral effect (Clβ):
- Negative (stabilizing) across most flight envelope
- Varies with alpha
```

### Roll Control

#### Ailerons (CLDA)

From AFWAL-TR-80-3141:

```
Deflection: ±15° differential
Effectiveness: 0.00079 per degree (low alpha)
              0.00020 per degree (α = 50°)
```

Roll control effectiveness degrades significantly at high alpha, typical of swept-wing aircraft.

#### Spoilers

Four independent sections provide primary roll control:

```
Left Inner/Outer:   -4° to +50°/+17°
Right Inner/Outer:  -4° to +50°/+17°

Roll authority: 0.0005 to 0.0008 per degree per section
```

Modern DLC modification (AFC 735):
- Inner spoilers: Full deflection (-4° to +50°) for DLC
- Outer spoilers: Limited deflection for roll control
- Improves approach control

#### Roll Damping (CLP)

```
CLP values:
α = 0°:   -0.40
α = 10°:  -0.20
α = 30°:  -0.27
```

### Wing Sweep Effect on Roll

Spoiler effectiveness factor:

```
Sweep    22°    30°    40°    50°    58°
Factor   1.00   0.93   0.83   0.69   0.00
```

Beyond 55° sweep, spoilers are ineffective for roll control.

### Adverse Yaw

Roll control inputs generate yawing moments through:
- Spoiler deployment (assymetric drag)
- Aileron deflection
- Modeled in yaw moment equations

## Yaw Moment (Cn)

### Directional Stability (CNBAS)

Basic yawing moment from sideslip:

```
CNBAS = f(α, β)

Weathercock stability (Cnβ):
- Positive (stabilizing) at low alpha
- Varies with alpha, can reverse at high alpha
```

### Rudder Control (CnDR)

Twin vertical stabilizers with rudders:

```
Deflection: ±30° each
Effectiveness: 0.0053 to 0.0056 per degree (low alpha)
              Reduces to near zero at α > 50°
```

The large twin tails provide powerful directional control at moderate angles of attack but lose effectiveness at extreme attitudes.

### Yaw Damping (CnR)

```
CnR values:
α = 0°:   0.57
α = 20°:  0.53
α = 40°: -2.05
```

Positive yaw damping at normal flight conditions. Can become destabilizing at very high alpha.

### Roll-Yaw Coupling

Significant coupling effects modeled:
- Roll rate induces yaw (CnP)
- Side force from roll rate (CyP)
- Critical for spin and departure characteristics

## Wing Sweep Effects

Variable geometry from 22° to 68° (with oversweep to 75° on ground).

### Aerodynamic Schedule

Automatic wing sweep system (CADC):

```
Mach     Commanded Sweep (flaps retracted)
0.45     22° (minimum)
0.71     22° (hold)
0.91     68° (maximum)
```

With flaps extended, wings locked at 22° regardless of speed.

### Lift Changes

Sweep reduces lift coefficient but increases wing loading efficiency at high speed:

```
CL Factor at α = 0°:
  22° sweep: 1.00
  50° sweep: 0.79
  58° sweep: 0.65
```

### Drag Changes

Sweep significantly reduces wave drag in transonic region but increases induced drag at low speed.

### Control Effectiveness

All control surfaces lose effectiveness with aft sweep:
- Spoilers most affected (zero effect at 58°)
- Stabilator moderately affected
- Rudders affected due to fuselage masking

### Flutter and Structural Limits

Wing sweep increases structural strength against bending:

```
G-Limit Factor = 1/cos⁴(sweep - 22°)

Sweep    22°    40°    60°
Factor   1.00   0.82   0.39
```

At maximum sweep, wings can withstand higher G loads before structural damage.

## High-Alpha Characteristics

### Departure Resistance

The F-14 exhibits good departure resistance due to:

1. **Strong Pitch-Down Moment**: Cm becomes increasingly negative above α = 15°
2. **Automatic Slats**: Deploy at high alpha to maintain flow attachment
3. **SAS (Stability Augmentation System)**: Provides artificial stability
4. **Large Twin Tails**: Maintain directional control

### Post-Stall Behavior

Beyond α = 20°:
- Lift continues to increase up to α ≈ 35°-40°
- Strong pitch-down moment aids recovery
- Roll control degraded but present
- Directional control progressively lost

### Spin Characteristics

Model includes full 360° alpha and beta data from AFWAL-TR-80-3141 allowing simulation of:
- Developed spins (flat and steep)
- Spin entry and recovery
- Inverted flight dynamics

## Ground Effects

### Touchdown Dynamics

Landing gear modeled with:
- Spring constant: 42,500 lbs/ft (main), 42,500 lbs/ft (nose)
- Damping: 2,000 lbs/ft/sec
- Friction: μs = 0.8, μd = 0.5, μr = 0.02

### Nose Gear Kneeling

Unique F-14 feature for carrier operations:
- Extends strut 1.87 ft for catapult attachment
- Compresses 1.17 ft for low profile on deck
- Modeled in f-14b-landing-gear.xml

### Structural Contact Points

Multiple contact points protect:
- Wing tips (left/right)
- Nose cone
- Canopy
- Front fuselage
- Rear fuselage
- Vertical tails

## Limitations and Assumptions

### Known Limitations

1. **Wing Sweep and Lateral Spoilers**: Based on CFD (OpenVSP) rather than wind tunnel data
2. **ACLS (Automatic Carrier Landing System)**: Not currently modeled
3. **Compressor Stall**: TF-30 engine specific, modeled separately in engine file
4. **Transonic Effects**: Some interpolation between subsonic and supersonic data

### Validated Ranges

The model is validated across:
- **Alpha**: -10° to +55° (primary data), extended to ±90° for departures
- **Beta**: ±20°
- **Mach**: 0.0 to 2.6
- **Altitude**: Sea level to 40,000 ft

### Future Improvements

Potential areas for enhanced fidelity:
1. More detailed ACLS implementation
2. Additional transonic drag data
3. Refined external stores aerodynamics
4. Enhanced vortex flow modeling at high alpha

## References

### Primary Sources

1. AFWAL-TR-80-3141 Parts I & III, Johnston et al., 1980
2. NASA TM-X-62306, Eckert & Maki, October 1973
3. NASA TM-X-62244, Eckert & Maki, August 1973
4. NASA TN D-6909, Grafton & Anglin, 1972
5. NASA-TM-81833, Kelly & Brown, May 1980
6. NASA-TM-81972, Kelly & Enevoldson, December 1981
7. NASA-TM-84643, Spearman, March 1983
8. NASA-TM-104326, Holzman et al., November 1996

### Computational Methods

- OpenVSP for wing sweep and spoiler CFD
- Jeff Miller, "Investigation of Transonic Drag Computations in APAS", 2002

### Online Resources

- FlightGear F-14 Model: http://zaretto.com/f-14
- Author: Richard Harrison (rjh@zaretto.com)

---

*This documentation describes the aerodynamic model as implemented in f-14a.xml and represents years of research and development to create a high-fidelity simulation suitable for both flight simulation and engineering analysis.*
