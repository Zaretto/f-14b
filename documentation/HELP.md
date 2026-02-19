Grumman F-14 Tomcat - Quick Reference
=====================================

For full documentation and contributors, see [README.md](README.md)

Website: http://zaretto.com/f-14

-------------------------------------------------------------

Notes
-----

The JSBSim aerodynamics model uses data from AFWAL-TR-80-3141,
NASA TM-X-62306, and other technical reports. The model is accurate
for low speed clean configurations; high speed and configurations
are based on additional reports.

Because of the realistic model, the aircraft is harder to fly than
those with simpler aerodynamic models. In high-G turns, watch AOA
to avoid stalling the wings or control surfaces.

The hydraulics and electrics require engine power or external power.
Loss of engines at low speed means loss of flight controls.

-------------------------------------------------------------

Engine Start
------------

1. Ensure sufficient fuel
2. Pull fuel cutoff valves (yellow striped handles on glareshield)
3. Select R Crank from engine control panel (next to throttles)
4. At 20% N2, push in cutoff for R engine
5. Repeat for Left engine

Alternative: Use quickstart from menu once fuel is loaded.

Once N2 reaches 18%, hydraulics power the backup electrical generator
and cockpit instruments illuminate.

-------------------------------------------------------------

Takeoff
-------

Shore based:
- MIL thrust, flaps as preferred
- Rotate at 130-160 kts depending on configuration
- Do not exceed 10 degrees pitch

Carrier based:
- Position aircraft onto catapult
- Kneel nosegear using cockpit switch
- Shift-L to engage launchbar
- Set power to MIL, flaps down
- Shift-C to launch
- Control pitch after leaving deck, retract gear
- Turn right or left depending on catapult

-------------------------------------------------------------

Keyboard Shortcuts
------------------

Flight Controls:
  Home/End      Elevator trim increase/decrease
  f/F           Flaps down/up
  </> 	        Wing sweep forward/aft
  =             Wing sweep auto mode
  o             Toggle arrester hook

Autopilot & SAS:
  Ctrl-t        Toggle AFCS Attitude Mode
  Ctrl-a        Enable AFCS Altitude Mode
  *             Engage AFCS Altitude Mode
  Ctrl-h        Enable AFCS Heading Mode
  Ctrl-p        Toggle Pitch SAS
  Ctrl-k        Toggle Roll SAS
  Ctrl-l        Toggle Yaw SAS

Landing:
  Ctrl-S        Toggle APC (Approach Power Compensator)
  Ctrl-d        Stick DLC/Chaff switch
  l/n           DLC thumbwheel fore/aft
  Ctrl-y        Toggle ground spoilers armed

Radar:
  Shift-E/R     Radar range decrease/increase
  q             Toggle Radar Standby Mode
  Ctrl-n        Toggle Radar Mode (Pulse/TWS)
  h             Cycle HSD modes (radar/compass/ECM)
  y             Next target
  Shift-y       Radar azimuth coverage
  i/I           Radar up/down
  D             Radar elevation coverage
  r             STT Lock
  U             Undesignate
  d             Pilot automatic lockon mode

Weapons:
  Ctrl-w        Cycle Master Arm modes
  m             Cycle Stick Weapon Selector
  Shift-m       Cycle pylons selection
  e             Fire selected weapon

Countermeasures:
  Ctrl-q        Release chaff/flare

Views & Misc:
  Ctrl-v        Toggle Pilot/RIO view
  Q             Reset view
  c             Toggle canopy and ladder
  u             Toggle refueling probe
  Ctrl-o        Toggle wing oversweep (ground only)
  S             Toggle smoke
  F6            Eject
  Ctrl-e        Ignore selected MP pilots

GCI (Ground Control Intercept):
  Ctrl-1        Picture request
  Ctrl-2        Bogey Dope request
  Ctrl-3        Cutoff request

HUD Multikey Shortcuts:
  :AHs          Toggle HUD on/off
  :AHt          HUD Takeoff mode
  :AHc          HUD Cruise mode
  :AHa          HUD A/A mode
  :AHg          HUD A/G mode
  :AHl          HUD Landing mode

-------------------------------------------------------------

Weapons Operation
-----------------

Available weapons:
- M61A1 Vulcan 20mm cannon (675 rounds)
- AIM-9 Sidewinder (heat-seeking)
- AIM-7 Sparrow (semi-active radar)
- AIM-54 Phoenix (active radar, long range)
- MK-83 bombs (with CCIP on HUD)

Load configurations available via Tomcat Controls > Fuel and Stores:
- FAD light: 4x AIM-9
- FAD: 2x AIM-9, 2x AIM-7, 2x AIM-54
- FAD heavy: Full missile loadout
- Bombcat: MK-83 bombs

Gun Operation:
1. Select HUD A/A Mode (:AHa)
2. Select Gun mode with weapon selector (m)
3. Switch Master Arm on (Ctrl-w)
4. Press e to fire

The HUD displays pipper, remaining rounds (G symbol), and closure
rate scale when a target is locked in TWS AUTO mode.

AIM-9 Sidewinder:
- Select SW mode with weapon selector (m)
- Master Arm on - seeker tone audible
- Acquire target - tone increases when locked
- Fire with e at 3-6 NM for best results
- Uses Proportional Navigation guidance

AIM-7 Sparrow:
- Semi-active radar homing
- Requires continuous radar illumination
- Select SP mode, lock target, fire

AIM-54 Phoenix:
- Long range active radar missile
- Can be fired in "maddog" mode (no lock)
- Sample guided until terminal phase
- Dynamic launch zone displayed on HUD

MK-83 Bombs:
- CCIP (Continuously Computed Impact Point) shown on HUD
- Pipper crosses out when insufficient time to arm

-------------------------------------------------------------

Autopilot Operation
-------------------

Attitude Hold Mode (Ctrl-t):
- Main autopilot mode, required for other modes
- Disengages with stick pressure, reengages at center
- Holds pitch ±30°, bank ±60°
- Disable for aerobatics and inverted flight

Altitude Mode:
1. Engage Attitude Mode (Ctrl-t)
2. Enable Altitude Mode (Ctrl-a) - AP REF light illuminates
3. Fly to desired altitude
4. Engage with * (asterisk)
CAUTION: Stabilize aircraft before engaging at high speed

Heading Mode:
1. Engage Attitude Mode (Ctrl-t)
2. Maneuver to desired heading
3. Enable Heading Mode (Ctrl-h) at <5° bank

SAS channels must be engaged (default). Toggle with Ctrl-p/k/l.

-------------------------------------------------------------

APC (Approach Power Compensator)
--------------------------------

Automatically regulates thrust for optimum approach AOA.

Prerequisites:
- Gear handle down
- Weight off wheels
- Throttles between 68-98% RPM

Toggle with Ctrl-S. Disengages when:
- Throttles to MIL (98%) or idle (68%)
- Gear handle raised
- Weight on wheels

AUTO THROT caution light illuminates for 10 seconds when disengaged.

-------------------------------------------------------------

DLC (Direct Lift Control)
-------------------------

Provides glidepath correction without changing power or AOA.
Spoilers and horizontal stabilizers move simultaneously.

Two implementations available (selectable per livery):
- Original: All spoilers -4° to +17° for roll and DLC
- AFC MOD 735: Inner spoilers for DLC (-4° to +50°),
               outer spoilers for roll only

Controls:
- Ctrl-d: DLC/Chaff switch
- l/n: DLC thumbwheel fore/aft

-------------------------------------------------------------

Ground Spoilers
---------------

Provide additional drag after touchdown.

1. Arm before landing (Ctrl-y)
2. After touchdown, pull throttle to idle
3. Spoilers deploy fully
4. Ctrl-y to disarm

-------------------------------------------------------------

Radar & Avionics
----------------

AWG-9 Radar:
- Pulse Search and TWS (Track While Scan) modes
- Toggle with Ctrl-n
- Range adjustment with Shift-E/R

RWR (Radar Warning Receiver):
- Displays threats on HSD in ECM mode
- Two threat levels indicated
- Cycle HSD modes with h

Datalink:
- Receives target data from compatible aircraft
- Contacts displayed beyond own radar range

IFF:
- Channel selection available
- Interrogates contacts for friend/foe

-------------------------------------------------------------

Multiplayer Features
--------------------

Dual Control:
- f-14b: Pilot aircraft (RIO can join)
- f14b-bs: Backseater mode for joining pilot

Emesary System:
- Weapons visible to other players
- Damage synchronized across MP
- Flare/chaff visible to others
- Persistent craters when damage enabled

-------------------------------------------------------------

For detailed technical documentation, see README.md
