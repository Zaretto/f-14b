# F-14 Aerodynamic Coefficient Buildup

**Author:** Richard Harrison (rjh@zaretto.com)
**Aircraft:** F-14A Tomcat with TF30-P-414 engines
**FDM:** JSBSim
**Documentation Date:** 2026-01-27

## Introduction

This document provides a comprehensive reference for the aerodynamic coefficient buildup used in the F-14 JSBSim flight dynamics model. The model is based on high-fidelity wind tunnel and flight test data from NASA and the Air Force, providing accurate representation of the F-14's aerodynamic characteristics across the full flight envelope.

### Reference Frame and Sign Conventions

The F-14 JSBSim model uses the standard body-axis system:

- **X-axis**: Forward (positive toward nose)
- **Y-axis**: Right wing (positive to starboard)
- **Z-axis**: Down (positive downward)

**Force Sign Conventions:**
- **Lift (L)**: Positive upward (perpendicular to velocity vector)
- **Drag (D)**: Positive aft (parallel to velocity vector)
- **Side Force (Y)**: Positive to starboard

**Moment Sign Conventions:**
- **Roll Moment (Cl)**: Positive right wing down
- **Pitch Moment (Cm)**: Positive nose up
- **Yaw Moment (Cn)**: Positive nose right

**Aerodynamic Reference Point (AERORP):** 419 inches aft of nose

### JSBSim Coefficient System

JSBSim computes aerodynamic forces and moments using dimensional equations:

```
LIFT  = qbar × S × Cl × Cmach × ClSweepFactor
DRAG  = qbar × S × Cd × ClSweepFactor
SIDE  = qbar × S × Cy
ROLL  = qbar × S × b × CL
PITCH = qbar × S × c × CM × Cmach
YAW   = qbar × S × b × CN
```

Where:
- `qbar` = dynamic pressure (psf)
- `S` = wing reference area (565 ft²)
- `b` = reference span (64 ft)
- `c` = mean aerodynamic chord (9.82 ft)

### Primary Data Sources

| Reference | Description | Application |
|-----------|-------------|-------------|
| **AFWAL-TR-80-3141 Part III** | Wind tunnel data at 22° sweep, clean, low speed | Core lift, drag, side force, and moment coefficients |
| **NASA TM-X-62306** | Lateral-directional characteristics in high-lift config | Flaps, slats, spoilers, landing gear increments |
| **NASA TM-X-62244** | Longitudinal characteristics in high-lift config | Additional flap and slat validation data |
| **NASA TM-81833** | Flight control system schematics | Stabilizer, DLC, and APC modeling |
| **NASA TN D-6909** | Variable sweep fighter data | Wing sweep factor tables |
| **NASA TM-84643** | Mach effects on wing-tail configurations | Mach number correction factor (Cmach) |
| **OpenVSP CFD** | Computational fluid dynamics | Wing sweep and lateral spoiler effects |

---

## 1. Lift Coefficient (Cl)

The lift coefficient is built up from a base value plus incremental contributions from control surfaces, high-lift devices, and dynamic effects.

### Total Buildup Formula

```
Cl = ClBAS + CldAuxFlaps + CldMainFlaps + CldSlats
     + CldSpoilersLeftInner + CldSpoilersLeftOuter
     + CldSpoilersRightInner + CldSpoilersRightOuter
     + CldLandingGear + ClDasym + CldSpeedbrake
```

**Final LIFT Force:**
```
LIFT = qbar × S × Cl × Cmach × ClSweepFactor
```

Note that lift is further modified by the `Cmach` factor and the `ClSweepFactor`.

### 1.1 Base Lift: ClBAS

**Source:** AFWAL-TR-80-3141 Part III - Figure 36
**Description:** Basic lift coefficient for clean configuration at 22° wing sweep

**Independent Variables:**
- Alpha (angle of attack): -55° to +55°
- Beta (sideslip angle): -20° to +20°

**Key Characteristics:**
- Symmetric about beta = 0° (as expected)
- Maximum Cl ≈ 1.93 at α = 35°, β = 0°
- Minimum Cl ≈ -1.93 at α = -35°, β = 0°
- Shows classic stall behavior above 35° AOA

**Sample Values (β = 0°):**

| Alpha (°) | ClBAS |
|-----------|-------|
| -55 | -1.427 |
| -35 | -1.930 |
| -20 | -1.603 |
| 0 | 0.078 |
| 10 | 1.013 |
| 20 | 1.603 |
| 35 | 1.930 |
| 45 | 1.669 |
| 55 | 1.427 |

### 1.2 Mach Factor: Cmach

**Source:** NASA-TM-84643 Figure 3 (Clalpha) and NASA-aaia-2000-0900 Figure 4
**Description:** Mach number correction factor applied to all coefficients except drag

This is a critical factor that modifies lift (and pitch moment) based on Mach number and angle of attack. It accounts for compressibility effects.

**Independent Variables:**
- Alpha: -20° to +50°
- Mach: 0.0 to 3.0

**Key Characteristics:**
- Unity (1.0) below Mach 0.2
- Transonic bump peaking at Mach 1.1 (~1.4-1.45)
- Supersonic decay beyond Mach 1.5
- AOA dependency shows higher factors at positive alpha

**Sample Values (α = 4°):**

| Mach | Cmach |
|------|-------|
| 0.0 | 1.000 |
| 0.6 | 0.958 |
| 0.9 | 1.230 |
| 1.1 | 1.405 |
| 1.5 | 1.124 |
| 2.1 | 0.762 |
| 3.0 | 0.512 |

### 1.3 Wing Sweep Factor: ClSweepFactor

**Source:** NASA-TM-D-6909
**Description:** Lift modification due to wing sweep relative to 22° baseline

**Independent Variables:**
- Alpha: 0° to 55°
- Wing sweep: 0° to 68°

**Key Characteristics:**
- Unity (1.0) at 22° sweep (baseline configuration)
- Reduced lift at high sweep angles at low AOA
- Near unity or increased lift at high sweep with high AOA

**Sample Values (α = 10°):**

| Sweep (°) | ClSweepFactor |
|-----------|---------------|
| 0 | 1.000 |
| 22 | 1.000 |
| 50 | 0.829 |
| 58 | 0.701 |

**Sample Values (α = 30°):**

| Sweep (°) | ClSweepFactor |
|-----------|---------------|
| 22 | 1.000 |
| 50 | 1.027 |
| 58 | 0.955 |

This shows the swept wing maintains better high-alpha performance.

### 1.4 Auxiliary Flaps: CldAuxFlaps

**Source:** NASA TM-X-62306 (derived, contributing 30% of total flap effect)
**Description:** Lift increment due to auxiliary flaps deflection

**Independent Variables:**
- Alpha: -5° to +29°
- Aux flap deflection: 0° to 35°

**Characteristics:**
- Peak effectiveness around α = 19° (ΔCl = 0.219 at 35° deflection)
- Minimal effect at extreme AOA

**Sample Values (35° flap deflection):**

| Alpha (°) | CldAuxFlaps |
|-----------|-------------|
| -5 | 0.183 |
| 1 | 0.160 |
| 7 | 0.153 |
| 15 | 0.193 |
| 19 | 0.219 |
| 29 | 0.180 |

### 1.5 Main Flaps: CldMainFlaps

**Source:** NASA TM-X-62306 (derived, contributing 70% of total flap effect)
**Description:** Lift increment due to main flaps deflection

**Independent Variables:**
- Alpha: -5° to +29°
- Main flap deflection: 0° to 35°

**Characteristics:**
- Peak effectiveness around α = 19° (ΔCl = 0.510 at 35° deflection)
- Significantly larger effect than aux flaps (70% vs 30% split)

**Sample Values (35° flap deflection):**

| Alpha (°) | CldMainFlaps |
|-----------|--------------|
| -5 | 0.428 |
| 1 | 0.373 |
| 7 | 0.357 |
| 15 | 0.451 |
| 19 | 0.510 |
| 29 | 0.421 |

**Combined Flap Effect (35° deflection at α = 19°):**
Total ΔCl = 0.219 + 0.510 = **0.729**

### 1.6 Slats: CldSlats

**Source:** NASA TM-X-62306 (derived, contributing 20% of total leading edge effect)
**Description:** Lift increment due to leading edge slat deflection

**Independent Variables:**
- Alpha: -5° to +29°
- Slat deflection: 0° to 17°

**Characteristics:**
- Peak effectiveness around α = 19° (ΔCl = 0.182 at 17° deflection)
- Extends stall AOA and improves high-alpha lift

**Sample Values (17° slat deflection):**

| Alpha (°) | CldSlats |
|-----------|----------|
| -5 | 0.153 |
| 1 | 0.133 |
| 7 | 0.127 |
| 15 | 0.161 |
| 19 | 0.182 |
| 29 | 0.150 |

### 1.7 Spoilers: CldSpoilersLeftInner/Outer, CldSpoilersRightInner/Outer

**Source:** NASA TM-X-62306 Test 2 Run 5,10
**Description:** Lift decrement due to spoiler deflection (always negative)

The F-14 has four spoiler groups: left/right inner/outer. Each group has identical per-degree effectiveness.

**Independent Variable:** Alpha: -5° to +29°
**Control Input:** Spoiler deflection (degrees)

**Characteristics:**
- All four groups have identical tables
- Lift loss decreases with increasing AOA
- Used for DLC (Direct Lift Control) and roll control

**Sample Values (per degree of deflection):**

| Alpha (°) | Lift Decrement |
|-----------|----------------|
| -5 | -0.00394 |
| 1 | -0.00493 |
| 5 | -0.00503 |
| 11 | -0.00492 |
| 15 | -0.00419 |
| 21 | -0.00233 |
| 29 | -0.00032 |

**Example:** 10° deflection of all four spoilers at α = 5°:
ΔCl = 4 × (-0.00503) × 10 = **-0.201**

### 1.8 Landing Gear: CldLandingGear

**Source:** NASA TM-X-62306 Test 4 Run 9, Run 10
**Description:** Lift increment due to landing gear extension

**Independent Variable:** Alpha: -5° to +29°
**Control Input:** Gear position (0 = up, 1 = down)

**Characteristics:**
- Generally negative (reduces lift) except at very low AOA
- Most significant around landing AOA (10-15°)

**Sample Values (gear down, multiplied by gear-pos-norm):**

| Alpha (°) | CldLandingGear |
|-----------|----------------|
| -5 | -0.0248 |
| 1 | 0.0012 |
| 5 | -0.0109 |
| 11 | -0.0070 |
| 15 | -0.0346 |
| 21 | -0.0283 |

### 1.9 Stabilizer Effects: ClDS1 and ClDS2

**Source:** AFWAL-TR-80-3141 Part III - Figures 38, 39
**Description:** Lift increments due to stabilizer deflection

The F-14 horizontal stabilizer contributes to total lift. Two coefficients define the stabilizer effectiveness:

#### ClDS1

**Per-degree effectiveness:**

| Alpha (°) | ClDS1 |
|-----------|-------|
| -65 | 0.0065 |
| 0 | 0.0165 |
| 20 | 0.0165 |
| 35 | 0.0115 |
| 55 | 0.0065 |

#### ClDS2

**Used for deflections beyond -10°:**

| Alpha (°) | ClDS2 |
|-----------|-------|
| -60 | -0.0055 |
| 0 | 0.0065 |
| 15 | 0.0125 |
| 35 | 0.0135 |
| 55 | 0.0055 |

**Application:**
```
DClDS1 = stabilizer-1p-factor × ClDS1 × elevator-pos-deg
DClDS2 = stabilizer-2p-factor × [-10 × ClDS1 + (stab-pos-deg + 10) × ClDS2]
```

These are scaled by the CADC control effectivity factor and then added to total lift.

### 1.10 Asymmetric Damage: ClDasym

**Source:** Model-specific damage modeling
**Description:** Lift reduction due to wing asymmetry from battle damage

```
ClDasym = wing-asymmetry² × ClBAS × (-0.3)
```

Where `wing-asymmetry` is -1, 0, or 1. The square removes sign so damaged wings always reduce lift.

### 1.11 Speedbrake: CldSpeedbrake

**Source:** NASA TM-X-62306 Test [15,18] [28,32]
**Description:** Lift increment due to speedbrake extension

**Independent Variable:** Alpha: -15° to +55°
**Control Input:** Speedbrake deflection (degrees)

**Characteristics:**
- Small positive or negative lift effects
- Most significant at high AOA

**Sample Values (per degree of deflection):**

| Alpha (°) | CldSpeedbrake |
|-----------|---------------|
| -15 | 0.00202 |
| 0 | -0.00036 |
| 10 | -0.00142 |
| 25 | 0.00084 |
| 35 | -0.00399 |
| 55 | -0.01621 |

---

## 2. Drag Coefficient (Cd)

The drag coefficient represents the total parasitic and induced drag of the aircraft.

### Total Buildup Formula

```
Cd = CdMach + CdBAS + CddLandingGear + CddSpeedbrake + CdTNK + CdDasym
     + CddAuxFlaps + CddMainFlaps + CddSlats
     + CddSpoilersLeftInner + CddSpoilersLeftOuter
     + CddSpoilersRightInner + CddSpoilersRightOuter
```

**Final DRAG Force:**
```
DRAG = qbar × S × Cd × ClSweepFactor
```

Note that drag is modified by `ClSweepFactor` but NOT by `Cmach`.

### 2.1 Base Drag: CdBAS

**Source:** AFWAL-TR-80-3141 Part III
**Description:** Basic drag coefficient for clean configuration at 22° sweep

**Independent Variables:**
- Alpha: -55° to +55°
- Beta: -20° to +20°

**Key Characteristics:**
- Minimum drag ~0.017 at low AOA
- Rapidly increasing drag with AOA
- Symmetric about β = 0°

**Sample Values (β = 0°):**

| Alpha (°) | CdBAS |
|-----------|-------|
| -55 | 2.035 |
| -30 | 1.113 |
| -10 | 0.168 |
| 0 | 0.025 |
| 10 | 0.168 |
| 20 | 0.577 |
| 30 | 1.113 |
| 40 | 1.505 |
| 50 | 1.890 |
| 55 | 2.035 |

### 2.2 Mach Drag: CdMach

**Source:** Model-specific (CFD-derived)
**Description:** Compressibility drag rise with Mach number and wing sweep

This is one of the most complex tables in the model, showing the dramatic transonic drag rise.

**Independent Variables:**
- Mach number: 0.2 to 2.5
- Alpha: -15° to +15°
- Wing sweep: 20°, 45°, 68° (3D table)

**Characteristics:**
- Zero below Mach 0.7
- Dramatic transonic drag rise peaking around Mach 0.94-1.05
- Reduced drag with increased wing sweep
- Scaled by tuning parameter `tuning/CMBRK`

**Sample Values (22° sweep, α = 0°):**

| Mach | CdMach |
|------|--------|
| 0.60 | 0.000 |
| 0.80 | 0.022 |
| 0.90 | 0.036 |
| 0.94 | 0.106 |
| 1.05 | 0.406 |
| 1.10 | 0.118 |
| 1.20 | 0.052 |
| 1.80 | 0.045 |

**Sample Values (68° sweep, α = 0°):**

| Mach | CdMach |
|------|--------|
| 0.80 | 0.010 |
| 0.94 | 0.045 |
| 1.05 | 0.113 |
| 1.10 | 0.046 |
| 1.20 | 0.044 |

Note the dramatic reduction in transonic drag rise with full aft sweep (68°).

### 2.3 External Tanks: CdTNK

**Source:** Model-specific
**Description:** Drag increment from external fuel tanks

**Independent Variable:** Mach number
**Control Input:** `fcs/external-tanks-fitted` (0 or 1)

**Sample Values (per tank set):**

| Mach | CdTNK |
|------|-------|
| 0.40-0.80 | 0.0005 |
| 0.85 | 0.0011 |
| 0.90 | 0.0065 |
| 1.00 | 0.0078 |
| 2.00 | 0.0081 |

The external tanks add significant transonic drag.

### 2.4 Landing Gear: CddLandingGear

**Source:** NASA TM-X-62306 Test 4 Run 9, Run 10
**Description:** Drag increment from extended landing gear

**Independent Variable:** Alpha: -5° to +29°
**Control Input:** `gear/gear-pos-norm` (0 to 1)

**Sample Values (gear down):**

| Alpha (°) | CddLandingGear |
|-----------|----------------|
| -5 | 0.0212 |
| 1 | 0.0212 |
| 9 | 0.0153 |
| 15 | 0.0093 |
| 21 | 0.0053 |
| 27 | -0.0051 |

Significant drag increase, especially at low AOA.

### 2.5 Speedbrake: CddSpeedbrake

**Source:** NASA TM-X-62306 Test [15,18] [28,32]
**Description:** Drag increment from speedbrake extension

**Independent Variable:** Alpha: -15° to +55°
**Control Input:** Speedbrake deflection (degrees)

**Sample Values (per degree):**

| Alpha (°) | CddSpeedbrake |
|-----------|---------------|
| -15 | 0.00111 |
| 0 | 0.00099 |
| 10 | 0.00073 |
| 20 | 0.00085 |
| 35 | 0.00116 |
| 55 | 0.00180 |

### 2.6 Flaps and Slats: CddAuxFlaps, CddMainFlaps, CddSlats

**Source:** NASA TM-X-62306
**Description:** Drag increments from high-lift devices

These follow similar patterns to the lift increments but represent the induced and parasitic drag from deflected surfaces.

#### CddAuxFlaps (35° deflection):

| Alpha (°) | CddAuxFlaps |
|-----------|-------------|
| -5 | 0.0176 |
| 5 | 0.0296 |
| 13 | 0.0345 |
| 21 | 0.0211 |
| 29 | 0.0441 |

#### CddMainFlaps (35° deflection):

| Alpha (°) | CddMainFlaps |
|-----------|--------------|
| -5 | 0.0411 |
| 5 | 0.0691 |
| 13 | 0.0805 |
| 21 | 0.0492 |
| 29 | 0.1029 |

#### CddSlats (17° deflection):

| Alpha (°) | CddSlats |
|-----------|----------|
| -5 | 0.0147 |
| 5 | 0.0247 |
| 13 | 0.0287 |
| 21 | 0.0176 |
| 29 | 0.0367 |

### 2.7 Spoilers: CddSpoilersLeftInner/Outer/RightInner/Outer

**Source:** NASA TM-X-62306 Test 2 Run 5,10
**Description:** Drag increment from spoiler deflection

All four spoiler groups share identical tables.

**Sample Values (per degree):**

| Alpha (°) | Drag Increment |
|-----------|----------------|
| -5 | 0.000403 |
| 1 | 0.000165 |
| 7 | -0.000022 |
| 13 | 0.000206 |
| 21 | -0.000424 |
| 29 | -0.000316 |

Spoilers can actually reduce drag at certain AOA (likely due to flow separation control).

### 2.8 Stabilizer Effects: CdDS1, CdDS2, DCdDS1, DCdDS2

**Source:** Model-specific
**Description:** Drag from stabilizer deflection

Similar to lift, two-part model for stabilizer drag:

#### CdDS1:

| Alpha (°) | CdDS1 |
|-----------|-------|
| -65 | 0.0140 |
| -5 | -0.0003 |
| 0 | -0.0012 |
| 15 | 0.0040 |
| 30 | 0.0095 |
| 55 | 0.0140 |

#### CdDS2:

| Alpha (°) | CdDS2 |
|-----------|-------|
| -65 | 0.0090 |
| 0 | -0.0033 |
| 20 | 0.0020 |
| 40 | 0.0095 |
| 55 | 0.0090 |

### 2.9 Asymmetric Damage: CdDasym

**Description:** Drag increase from wing asymmetry

```
CdDasym = wing-asymmetry² × CdBAS × (-0.3)
```

Actually reduces drag coefficient (but with increased induced drag from asymmetric lift).

---

## 3. Side Force Coefficient (Cy)

The side force coefficient represents lateral aerodynamic force, primarily from sideslip.

### Total Buildup Formula

```
Cy = CyBAS + CyP + CyR + CyDA + CyDR
```

**Final SIDE Force:**
```
SIDE = qbar × S × Cy
```

### 3.1 Base Side Force: CyBAS

**Source:** AFWAL-TR-80-3141
**Description:** Side force from sideslip angle

**Independent Variables:**
- Alpha: -20° to +55°
- Beta: -20° to +20°

**Characteristics:**
- Linear with beta at most AOA
- Anti-symmetric (sign change with beta)
- Reduced at high AOA (weakening vertical tail effectiveness)

**Sample Values (α = 0°):**

| Beta (°) | CyBAS |
|----------|-------|
| -20 | 0.294 |
| -10 | 0.146 |
| -5 | 0.064 |
| 0 | 0.000 |
| 5 | -0.064 |
| 10 | -0.146 |
| 20 | -0.294 |

**Sample Values (α = 40°):**

| Beta (°) | CyBAS |
|----------|-------|
| -20 | 0.257 |
| -10 | 0.145 |
| 0 | 0.000 |
| 10 | -0.145 |
| 20 | -0.257 |

### 3.2 Roll Rate Effect: CyP

**Source:** Model-specific
**Description:** Side force from roll rate

```
CyP = (b/2V) × p × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CyP coefficient |
|-----------|-----------------|
| -20 | -0.160 |
| -5 | 0.000 |
| 0 | 0.149 |
| 10 | 0.125 |
| 30 | 0.680 |
| 40 | 0.932 |
| 50 | -0.696 |

Shows complex high-alpha behavior with roll rate coupling.

### 3.3 Yaw Rate Effect: CyR

**Source:** Model-specific
**Description:** Side force from yaw rate

```
CyR = (b/2V) × r × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CyR coefficient |
|-----------|-----------------|
| -20 | 0.820 |
| 0 | 0.568 |
| 15 | 0.678 |
| 30 | -0.734 |
| 40 | -2.053 |

Strong damping at low AOA, becomes destabilizing at high AOA.

### 3.4 Aileron Effect: CyDA

**Source:** Model-specific
**Description:** Side force from aileron deflection

```
CyDA = left-aileron-pos-deg × cadc-effectivity × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CyDA (per deg) |
|-----------|----------------|
| -5 | -0.00095 |
| 5 | -0.00100 |
| 20 | -0.00030 |
| 35 | 0.00160 |
| 50 | 0.00255 |

### 3.5 Rudder Effect: CyDR

**Source:** Model-specific
**Description:** Primary lateral control via rudder

**Independent Variables:**
- Alpha: -10° to +55°
- Beta: -20° to +20°

**Sample Values (β = 0°):**

| Alpha (°) | CyDR (per deg) |
|-----------|----------------|
| 0 | 0.00560 |
| 10 | 0.00510 |
| 20 | 0.00420 |
| 30 | 0.00310 |
| 40 | 0.00150 |
| 50 | 0.00000 |

Rudder effectiveness decreases significantly at high AOA.

---

## 4. Roll Moment Coefficient (CL)

The roll moment coefficient produces rolling motion about the aircraft centerline.

### Total Buildup Formula

```
CL = CLBAS + CLP + CLR + CLDA + CLDR + CLDasym
     + CMLdSpoilersLeftInner + CMLdSpoilersLeftOuter
     + CMLdSpoilersRightInner + CMLdSpoilersRightOuter
```

**Final ROLL Moment:**
```
ROLL = qbar × S × b × CL
```

### 4.1 Base Roll Moment: CLBAS

**Source:** AFWAL-TR-80-3141
**Description:** Rolling moment from sideslip (dihedral effect)

**Independent Variables:**
- Alpha: -45° to +55°
- Beta: -20° to +20°

**Characteristics:**
- Generally positive dihedral effect (positive beta causes negative roll)
- Reversed at very low AOA
- Complex high-alpha behavior

**Sample Values (α = 0°):**

| Beta (°) | CLBAS |
|----------|-------|
| -20 | 0.0095 |
| -10 | 0.0060 |
| 0 | 0.0000 |
| 10 | -0.0060 |
| 20 | -0.0095 |

**Sample Values (α = 20°):**

| Beta (°) | CLBAS |
|----------|-------|
| -20 | 0.0550 |
| -10 | 0.0270 |
| 0 | 0.0000 |
| 10 | -0.0270 |
| 20 | -0.0550 |

### 4.2 Roll Damping: CLP

**Source:** Model-specific
**Description:** Roll damping from roll rate

```
CLP = (b/2V) × p × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CLP coefficient |
|-----------|-----------------|
| -10 | -0.410 |
| 0 | -0.402 |
| 10 | -0.200 |
| 25 | -0.200 |
| 40 | -0.186 |
| 55 | -0.089 |

Strong roll damping at all AOA, decreasing at high alpha.

### 4.3 Roll from Yaw Rate: CLR

**Source:** Model-specific
**Description:** Roll-yaw coupling

```
CLR = (b/2V) × r × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CLR coefficient |
|-----------|-----------------|
| -5 | -0.052 |
| 0 | 0.008 |
| 10 | 0.084 |
| 30 | 0.320 |
| 45 | 0.073 |
| 55 | -0.588 |

Shows strong coupling at moderate-high AOA.

### 4.4 Aileron Control: CLDA

**Source:** Model-specific
**Description:** Primary roll control via ailerons

**Independent Variables:**
- Alpha: 0° to +55°
- Beta: -20° to +20°

**Sample Values (β = 0°):**

| Alpha (°) | CLDA (per deg) |
|-----------|----------------|
| 0 | 0.000790 |
| 10 | 0.000790 |
| 20 | 0.000755 |
| 30 | 0.000485 |
| 40 | 0.000120 |
| 50 | 0.000220 |

Aileron effectiveness significantly reduced at high AOA.

### 4.5 Spoiler Roll Control: CMLdSpoilersLeftInner/Outer/RightInner/Outer

**Source:** Model-specific (CFD-derived)
**Description:** Roll moment from lateral spoiler deflection

The F-14 uses differential spoilers for roll control, especially at high sweep angles. Inner and outer spoilers have different effectiveness.

#### Inner Spoilers (per degree):

| Alpha (°) | Left Inner | Right Inner |
|-----------|------------|-------------|
| -15 | -0.000551 | +0.000551 |
| 0 | -0.000503 | +0.000503 |
| 10 | -0.000129 | +0.000129 |
| 20 | -0.000025 | +0.000025 |
| 35 | -0.000013 | +0.000013 |
| 55 | +0.000049 | -0.000049 |

#### Outer Spoilers (per degree):

| Alpha (°) | Left Outer | Right Outer |
|-----------|------------|-------------|
| -15 | -0.000761 | +0.000761 |
| 0 | -0.000695 | +0.000695 |
| 10 | -0.000178 | +0.000178 |
| 20 | -0.000035 | +0.000035 |
| 35 | -0.000018 | +0.000018 |

Outer spoilers provide approximately 38% more roll effectiveness than inner spoilers.

**Spoiler Effectiveness Factor:** `CSPL`

The spoiler roll effectiveness varies with wing sweep:

| Sweep (°) | CSPL Factor |
|-----------|-------------|
| 22 | 1.000 |
| 30 | 0.934 |
| 45 | 0.763 |
| 55 | 0.619 |
| 58 | 0.000 |

Spoilers become ineffective above 58° sweep (oversweep position).

### 4.6 Rudder Roll Effect: CLDR

**Source:** Model-specific
**Description:** Roll moment from rudder (adverse/proverse yaw)

**Independent Variables:**
- Alpha: 0° to +55°
- Beta: -20° to +20°

**Sample Values (β = 0°):**

| Alpha (°) | CLDR (per deg) |
|-----------|----------------|
| 0 | 0.000112 |
| 15 | 0.000094 |
| 30 | 0.000062 |
| 45 | 0.000010 |
| 50+ | 0.000000 |

### 4.7 Asymmetric Damage: CLDasym

**Description:** Roll moment from wing asymmetry

```
CLDasym = wing-asymmetry × ClSweepFactor × 0.076
```

Creates constant rolling moment requiring aileron trim.

---

## 5. Pitch Moment Coefficient (CM)

The pitch moment coefficient produces pitching motion about the lateral axis.

### Total Buildup Formula

```
CM = CMBAS + DCMB0 + CMQ + DCMDS1 + DCMDS2
     + CMMdLandingGear + CMMdSpeedbrake + DCMAIM
     + CMMdAuxFlaps + CMMdMainFlaps + CMMdSlats
     + CMMdSpoilersLeftInner + CMMdSpoilersLeftOuter
     + CMMdSpoilersRightInner + CMMdSpoilersRightOuter
```

**Final PITCH Moment:**
```
PITCH = qbar × S × c × CM × Cmach
```

Note that pitch moment is modified by `Cmach` factor.

### 5.1 Base Pitch Moment: CMBAS

**Source:** AFWAL-TR-80-3141
**Description:** Basic pitching moment about AERORP

**Independent Variables:**
- Alpha: -15° to +55°
- Beta: -20° to +20°

**Characteristics:**
- Stable (negative slope with alpha) at low AOA
- Becomes unstable at high AOA
- Relatively insensitive to beta

**Sample Values (β = 0°):**

| Alpha (°) | CMBAS |
|-----------|-------|
| -15 | 0.187 |
| -5 | 0.104 |
| 0 | 0.072 |
| 5 | 0.039 |
| 10 | 0.005 |
| 15 | -0.044 |
| 20 | -0.057 |
| 25 | -0.141 |
| 30 | -0.224 |
| 35 | -0.300 |
| 40 | -0.402 |
| 45 | -0.474 |
| 50 | -0.563 |
| 55 | -0.639 |

Negative values indicate nose-down moment (statically stable).

### 5.2 Base Offset: DCMB0

**Description:** Constant pitch moment offset

```
DCMB0 = -0.055
```

Applied at all alpha values, provides baseline nose-down trim.

### 5.3 Pitch Damping: CMQ

**Source:** Model-specific
**Description:** Pitch damping from pitch rate

```
CMQ = (c/2V) × q × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CMQ coefficient |
|-----------|-----------------|
| -20 | -14.1 |
| 0 | -15.4 |
| 10 | -18.0 |
| 20 | -21.1 |
| 30 | -15.2 |
| 40 | -7.7 |
| 50 | -13.1 |

Strong pitch damping at all AOA.

### 5.4 Stabilizer Control: CMDS1, CMDS2, DCMDS1, DCMDS2

**Source:** Model-specific
**Description:** Primary pitch control via all-moving horizontal stabilizer

#### CMDS1 (per degree):

| Alpha (°) | CMDS1 |
|-----------|-------|
| -55 | -0.0120 |
| -25 | -0.0160 |
| 0 | -0.0090 |
| 20 | -0.0160 |
| 40 | -0.0145 |
| 55 | -0.0120 |

#### CMDS2 (per degree for second regime):

| Alpha (°) | CMDS2 |
|-----------|-------|
| -55 | 0.0000 |
| -20 | -0.0170 |
| 0 | -0.0195 |
| 20 | -0.0170 |
| 40 | 0.0040 |
| 55 | 0.0000 |

The stabilizer uses a two-part model for different deflection ranges.

### 5.5 AIM Missiles: DCMAIM

**Source:** Model-specific
**Description:** Pitch moment from external AIM missiles

**Independent Variables:**
- Mach: 0.8 to 0.9
- Alpha: -6° to +20°

**Characteristics:**
- Nose-down at negative alpha
- Nose-up at positive alpha
- Most significant in transonic regime

**Sample Values (Mach 0.9):**

| Alpha (°) | DCMAIM |
|-----------|--------|
| -6 | -0.032 |
| -2 | -0.020 |
| 0 | -0.002 |
| 4 | 0.007 |
| 10 | 0.006 |

### 5.6 Landing Gear: CMMdLandingGear

**Source:** NASA TM-X-62306 Test 4 Run 9, Run 10
**Description:** Pitch moment from extended landing gear

**Sample Values (gear down):**

| Alpha (°) | CMMdLandingGear |
|-----------|-----------------|
| -5 | -0.0047 |
| 1 | -0.0125 |
| 7 | -0.0076 |
| 13 | -0.0153 |
| 19 | -0.0096 |
| 25 | 0.0057 |

Nose-down moment at landing AOA, helps prevent ballooning.

### 5.7 Speedbrake: CMMdSpeedbrake

**Source:** NASA TM-X-62306 Test [15,18] [28,32]
**Description:** Pitch moment from speedbrake

**Sample Values (per degree):**

| Alpha (°) | CMMdSpeedbrake |
|-----------|----------------|
| -15 | -0.00107 |
| 0 | 0.00014 |
| 15 | 0.00173 |
| 25 | 0.00049 |
| 45 | 0.00261 |

Generally nose-up (helps pitch up during approach).

### 5.8 Flaps and Slats: CMMdAuxFlaps, CMMdMainFlaps, CMMdSlats

**Source:** NASA TM-X-62306
**Description:** Pitch moments from high-lift devices

All three produce strong nose-down moments when deployed, requiring stabilizer trim.

#### CMMdAuxFlaps (35° deflection):

| Alpha (°) | CMMdAuxFlaps |
|-----------|--------------|
| -5 | -0.0170 |
| 1 | -0.0241 |
| 9 | -0.0036 |
| 15 | 0.0034 |
| 23 | -0.0339 |
| 29 | -0.0342 |

#### CMMdMainFlaps (35° deflection):

| Alpha (°) | CMMdMainFlaps |
|-----------|---------------|
| -5 | -0.0398 |
| 1 | -0.0563 |
| 9 | -0.0083 |
| 15 | 0.0079 |
| 23 | -0.0791 |
| 29 | -0.0797 |

#### CMMdSlats (17° deflection):

| Alpha (°) | CMMdSlats |
|-----------|-----------|
| -5 | -0.0142 |
| 1 | -0.0201 |
| 9 | -0.0030 |
| 15 | 0.0028 |
| 23 | -0.0283 |
| 29 | -0.0285 |

### 5.9 Spoilers: CMMdSpoilersLeftInner/Outer/RightInner/Outer

**Source:** NASA TM-X-62306 Test 2 Run 5,10
**Description:** Pitch moment from spoiler deflection

All four spoiler groups produce nose-up pitching moment (positive CM), which is used for Direct Lift Control (DLC).

**Sample Values (per degree, all four groups identical):**

| Alpha (°) | Pitch Increment |
|-----------|-----------------|
| -5 | 0.00148 |
| 1 | 0.00149 |
| 7 | 0.00160 |
| 13 | 0.00223 |
| 19 | 0.00098 |
| 25 | 0.00132 |

**DLC Application:**
When all four spoiler groups are deflected 10°:
ΔCM = 4 × 0.00160 × 10 = **0.064** (at α = 7°)

This provides significant pitch control for carrier approaches.

---

## 6. Yaw Moment Coefficient (CN)

The yaw moment coefficient produces yawing motion about the vertical axis.

### Total Buildup Formula

```
CN = CNBAS + DCNDS + CNP + CNR + CNDA + CNDR + CNDasym
     + CMNdSpoilersLeftInner + CMNdSpoilersLeftOuter
     + CMNdSpoilersRightInner + CMNdSpoilersRightOuter
```

**Final YAW Moment:**
```
YAW = qbar × S × b × CN
```

### 6.1 Base Yaw Moment: CNBAS

**Source:** AFWAL-TR-80-3141
**Description:** Weathercock stability (yaw from sideslip)

**Independent Variables:**
- Alpha: -10° to +55°
- Beta: -20° to +20°

**Characteristics:**
- Stable (negative CN for positive beta) at low AOA
- Becomes unstable above α ≈ 15°
- Maximum instability around α = 30-35°

**Sample Values (α = 0°):**

| Beta (°) | CNBAS |
|----------|-------|
| -20 | -0.040 |
| -10 | -0.022 |
| -5 | -0.010 |
| 0 | 0.000 |
| 5 | 0.010 |
| 10 | 0.022 |
| 20 | 0.040 |

**Sample Values (α = 30°, showing instability):**

| Beta (°) | CNBAS |
|----------|-------|
| -20 | 0.038 |
| -10 | 0.018 |
| 0 | 0.000 |
| 10 | -0.018 |
| 20 | -0.038 |

Sign reversal indicates directional instability.

### 6.2 Stabilizer Yaw Effect: DCNDS

**Description:** Yaw moment from stabilizer deflection

```
DCNDS = elevator-pos-deg × cadc-effectivity × sin(82×beta) × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | DCNDS base |
|-----------|------------|
| 0 | 0.000117 |
| 15 | 0.000183 |
| 30 | 0.000267 |
| 45 | 0.000050 |

### 6.3 Roll Rate Coupling: CNP

**Source:** AFWAL, Revised TM-81833 Fig37
**Description:** Yaw moment from roll rate

```
CNP = (b/2V) × p × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CNP coefficient |
|-----------|-----------------|
| -10 | 0.010 |
| 0 | -0.033 |
| 5 | -0.095 |
| 15 | -0.119 |
| 25 | -0.080 |
| 35 | 0.142 |
| 50 | -0.096 |

Strong adverse yaw at moderate AOA, becomes proverse at high AOA.

### 6.4 Yaw Damping: CNR

**Description:** Yaw damping from yaw rate

```
CNR = (b/2V) × r × [table(alpha)]
```

**Sample Values:**

| Alpha (°) | CNR coefficient |
|-----------|-----------------|
| -5 | -0.115 |
| 0 | -0.115 |
| 10 | -0.138 |
| 20 | -0.190 |
| 35 | -0.291 |
| 50 | 0.046 |

Strong yaw damping except at extreme AOA.

### 6.5 Aileron Yaw Effect: CNDA

**Description:** Adverse yaw from aileron deflection

**Sample Values:**

| Alpha (°) | CNDA (per deg) |
|-----------|----------------|
| 0 | 0.000250 |
| 10 | 0.000175 |
| 20 | -0.000050 |
| 30 | -0.000375 |
| 45 | -0.000825 |

Positive = adverse yaw at low AOA, becomes proverse at high AOA.

### 6.6 Rudder Control: CNDR

**Description:** Primary directional control

**Independent Variables:**
- Alpha: -5° to +55°
- Beta: -20° to +20°

**Sample Values (β = 0°):**

| Alpha (°) | CNDR (per deg) |
|-----------|----------------|
| -5 | -0.00158 |
| 0 | -0.00158 |
| 10 | -0.00144 |
| 20 | -0.00134 |
| 30 | -0.00077 |
| 40 | -0.00029 |
| 50+ | 0.00000 |

Strong rudder effectiveness at low AOA, degrading significantly above 30°.

### 6.7 Spoiler Yaw Effects: CMNdSpoilersLeftInner/Outer/RightInner/Outer

**Description:** Yaw moments from differential spoiler deployment

Spoilers create yaw moments, which must be coordinated with rudder for clean roll control.

#### Inner Spoilers (per degree):

| Alpha (°) | Left Inner | Right Inner |
|-----------|------------|-------------|
| -15 to -5 | -0.000006 | +0.000006 |
| 0 | 0.000071 | -0.000071 |
| 10 | 0.000138 | -0.000138 |
| 25 | 0.000149 | -0.000149 |
| 45 | 0.000146 | -0.000146 |
| 55 | 0.000143 | -0.000143 |

#### Outer Spoilers (per degree):

| Alpha (°) | Left Outer | Right Outer |
|-----------|------------|-------------|
| 0 | 0.000098 | -0.000098 |
| 10 | 0.000190 | -0.000190 |
| 25 | 0.000205 | -0.000205 |
| 45 | 0.000202 | -0.000202 |

Outer spoilers produce approximately 38% more yaw effect than inner spoilers.

### 6.8 Asymmetric Damage: CNDasym

**Description:** Constant yaw moment from wing asymmetry

```
CNDasym = wing-asymmetry × 0.01293512
```

Creates constant yaw moment requiring rudder trim.

---

## Summary Tables

### Primary Coefficient Sources

| Coefficient | Primary Source | Secondary Sources | Notes |
|-------------|----------------|-------------------|-------|
| ClBAS | AFWAL-TR-80-3141 Fig 36 | - | 22° sweep baseline |
| CdBAS | AFWAL-TR-80-3141 | - | Clean config |
| CdMach | CFD (OpenVSP) | - | 3D table with sweep |
| CyBAS | AFWAL-TR-80-3141 | - | Sideslip stability |
| CLBAS | AFWAL-TR-80-3141 | - | Dihedral effect |
| CMBAS | AFWAL-TR-80-3141 | - | Static stability |
| CNBAS | AFWAL-TR-80-3141 | - | Weathercock stability |
| Flaps/Slats | NASA TM-X-62306 | TM-X-62244 | 80/20 main/aux split |
| Spoilers | NASA TM-X-62306 | - | Four-group model |
| Landing Gear | NASA TM-X-62306 Test 4 | - | Run 9, 10 |
| Speedbrake | NASA TM-X-62306 | - | Tests 15,18,28,32 |
| ClSweepFactor | NASA TN D-6909 | - | Variable geometry |
| Cmach | NASA TM-84643 | NASA-aaia-2000-0900 | Compressibility |

### Coefficient Buildup Summary

| Axis | Base | High-Lift | Spoilers | Gear | Controls | Factors | Dynamic |
|------|------|-----------|----------|------|----------|---------|---------|
| **LIFT** | ClBAS | Flaps, Slats | 4 groups | Yes | Stabilizer | Cmach, Sweep | Speedbrake, Damage |
| **DRAG** | CdBAS | Flaps, Slats | 4 groups | Yes | Stabilizer | Sweep only | Mach, Tanks, Speedbrake |
| **SIDE** | CyBAS | - | - | - | Aileron, Rudder | - | CyP, CyR |
| **ROLL** | CLBAS | - | 4 groups | - | Aileron, Rudder | Sweep (indirect) | CLP, CLR, Damage |
| **PITCH** | CMBAS | Flaps, Slats | 4 groups | Yes | Stabilizer | Cmach | CMQ, AIM, Speedbrake |
| **YAW** | CNBAS | - | 4 groups | - | Aileron, Rudder, Stab | - | CNP, CNR, Damage |

### High-Lift Device Contribution Splits

Based on NASA TM-X-62306 with 35° flaps and 17° slats:

| Component | Lift % | Drag % | Pitch % |
|-----------|--------|--------|---------|
| Main Flaps | 70% | 70% | 70% |
| Aux Flaps | 30% | 30% | 30% |
| **Total Flaps** | **100%** | **100%** | **100%** |
| Slats | 20% | 20% | 20% |

At approach configuration (α = 15°, full flaps/slats):
- **Total ΔCl:** ≈ 0.81
- **Total ΔCd:** ≈ 0.18
- **Total ΔCm:** ≈ -0.15 (nose-down)

### Spoiler Effectiveness

| Group | Roll Authority | DLC Authority | Yaw Coupling |
|-------|----------------|---------------|--------------|
| Inner Left/Right | 100% | 100% | Lower |
| Outer Left/Right | 138% | 138% | Higher |

**DLC Configuration:** All four spoiler groups used together
**Roll Configuration:** Differential deflection, scaled by wing sweep

**Sweep Factor (CSPL) at critical angles:**
- 22°: 100% (full authority)
- 45°: 76% (reduced)
- 58°: 0% (disabled in oversweep)

### Control Surface Effectiveness at High AOA

| Surface | α = 0° | α = 20° | α = 40° | Degradation |
|---------|--------|---------|---------|-------------|
| Ailerons (CLDA) | 0.00079 | 0.00076 | 0.00012 | 85% loss |
| Rudder (CyDR) | 0.00560 | 0.00420 | 0.00150 | 73% loss |
| Rudder (CNDR) | -0.00158 | -0.00134 | -0.00029 | 82% loss |
| Stabilizer (CMDS1) | -0.0090 | -0.0160 | -0.0145 | Actually increases |
| Spoilers (Roll) | -0.00070 | -0.00003 | -0.00002 | 97% loss |

The F-14's horizontal stabilizer is the only primary control that maintains (and increases) effectiveness at high AOA.

### Transonic Drag Rise

Peak drag coefficient increments at Mach 0.94:

| Configuration | CdMach Peak | Notes |
|---------------|-------------|-------|
| 22° Sweep, α=0° | 0.106 | Baseline wing |
| 22° Sweep, α=10° | 0.140 | Increased with lift |
| 45° Sweep, α=0° | 0.044 | 58% reduction |
| 68° Sweep, α=0° | 0.045 | Further swept |

Wing sweep dramatically reduces transonic drag, enabling supersonic flight.

---

## References

1. **AFWAL-TR-80-3141 Part I**: Investigation of High-Angle-of-Attack Maneuver-limiting factors, Donald E. Johnston et al., 1980
2. **AFWAL-TR-80-3141 Part III**: Appendices aerodynamic models, Donald E. Johnston et al., 1980
3. **NASA TM-X-62244**: Low-speed wind-tunnel investigation of the longitudinal characteristics, William T. Eckert and Ralph L. Maki, August 1973
4. **NASA TM-X-62306**: Low-speed wind tunnel investigation of the lateral-directional characteristics, William T. Eckert and Ralph L. Maki, October 1973
5. **NASA TN D-6909**: Dynamic Stability Derivatives at Angles of Attack from -5° to 90°, Sue B. Grafton and Ernie L. Anglin, 1972
6. **NASA TM-81833**: Simulator results of an F-14A airplane utilizing an aileron-rudder interconnect, Kelly & Brown, May 1980
7. **NASA TM-84643**: Effects of Wing and tail location on aerodynamic characteristics, M. Leroy Spearman, March 1983
8. **NASA TM-104326**: Flight and Static Exhaust Flow Properties of an F110-GE-129 Engine, Holzman et al., November 1996
9. **NASA-aaia-2000-0900**: Aerodynamic characteristics, database development and flight simulation of the X-34 vehicle, Pamadi et al., 2012

---

## Appendix: Coefficient Naming Conventions

### JSBSim Coefficient Nomenclature

- **Cl** = Lift coefficient (lowercase 'l')
- **Cd** = Drag coefficient
- **Cy** = Side force coefficient
- **CL** = Roll moment coefficient (uppercase 'L')
- **CM** = Pitch moment coefficient (uppercase 'M')
- **CN** = Yaw moment coefficient (uppercase 'N')

### Incremental Coefficient Naming

- **Base**: e.g., `ClBAS`, `CdBAS` - fundamental coefficient from tables
- **Delta (D prefix)**: e.g., `DCdDS1` - change due to control deflection
- **Incremental (lowercase 'd' infix)**: e.g., `CldMainFlaps`, `CddLandingGear` - additive contributions
- **Dynamic**: e.g., `CLP`, `CMQ` - rate-dependent terms

### Control Surface Suffixes

- **DA**: Aileron deflection
- **DR**: Rudder deflection
- **DS**, **DS1**, **DS2**: Stabilizer deflection (dual regime)
- **Dasym**: Asymmetric damage effects
- **dMainFlaps**, **dAuxFlaps**, **dSlats**: High-lift devices
- **dLandingGear**: Landing gear
- **dSpeedbrake**: Speed brake
- **dSpoilersLeftInner/Outer/RightInner/Outer**: Lateral spoiler groups

### Dynamic Term Suffixes

- **P**: Roll rate (velocities/p-aero-rad_sec)
- **Q**: Pitch rate (velocities/q-aero-rad_sec)
- **R**: Yaw rate (velocities/r-aero-rad_sec)

### Modifier Functions

- **Cmach**: Mach number correction factor (lift and pitch moment only)
- **ClSweepFactor**: Wing sweep correction factor (lift and drag)
- **CSPL**: Spoiler effectiveness with wing sweep

---

**End of Document**
