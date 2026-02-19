# How Instruments and Flight Controls Move in FlightGear

*Traced through the [F-14B codebase](https://github.com/Zaretto/f-14b) as a worked example*

---

**The property tree is the bus. Everything communicates through it.**

There are three things that write to it and one thing that reads from it to move geometry on screen.

## 1. The FDM (JSBSim) - Table-Driven XML Processed by C++

The pilot moves the stick. FlightGear writes a normalised value (-1 to +1) into `fcs/elevator-cmd-norm`. From there it's entirely within JSBSim (C++ engine executing XML-defined systems).

The FCS sums stick + trim, then does a table lookup to convert normalised stick position to demanded stabilator deflection in degrees (-33 to +10):

[Systems/f-14-fcs.xml Lines 321-346](https://github.com/Zaretto/f-14b/blob/release/1.12/Systems/f-14-fcs.xml#L321-L346)
```xml
<summer name="fcs/pitch-cmd-norm">
    <input>fcs/pitch-trim-cmd-norm</input>
    <input>fcs/elevator-cmd-norm</input>
</summer>
<fcs_function name="fcs/pitch-1-dh-deg-v2">
    <function>
        <table>
            <independentVar lookup="row">fcs/pitch-cmd-norm</independentVar>
            <tableData>
                -1.000  -33.000
                 0.000    0.000
                 1.000   10.000
            </tableData>
        </table>
    </function>
</fcs_function>
```

That goes through the SAS pitch damper (lead-lag filter on pitch rate, [line 357](https://github.com/Zaretto/f-14b/blob/release/1.12/Systems/f-14-fcs.xml#L357)), gets summed with the AFCS autopilot command and flap/slat trim compensation, checked against hydraulic pressure (if hydraulics fail the surface holds its last position, [line 457](https://github.com/Zaretto/f-14b/blob/release/1.12/Systems/f-14-fcs.xml#L457)), and finally rate-limited through a kinematic element at 30 deg/sec to simulate the actuator:

[Systems/f-14-fcs.xml Lines 1021-1036](https://github.com/Zaretto/f-14b/blob/release/1.12/Systems/f-14-fcs.xml#L1021-L1036)
```xml
<kinematic name="fcs/elevator-pos-deg">
    <input>fcs/elevator-pos-demand-deg</input>
    <traverse>
        <setting><position>-33</position><time>1.75</time></setting>
        <setting><position>12</position><time>0.90</time></setting>
    </traverse>
    <output>fcs/elevator-pos-deg</output>
</kinematic>
```

The output property `fcs/elevator-pos-deg` is then used by the aero model's coefficient tables. Those coefficients are also table-driven - 2D lookups from NASA/AFWAL windtunnel data indexed by angle of attack, Mach, and surface deflection. JSBSim integrates the resulting forces and moments and writes the final normalised surface positions out to `surface-positions/`.

None of this is Nasal. It's all XML tables and transfer functions executed by the JSBSim C++ engine each FDM frame.

## 2. Nasal Scripts - Computed Properties

For instruments and systems that need logic beyond what JSBSim provides, Nasal scripts read properties, compute values, and write results back to the property tree.

The main loop is driven by Emesary frame notifications. Every frame, `instruments_exec` receives a `FrameNotification` and dispatches updates on alternating frames:

[Nasal/instruments.nas Lines 551-670](https://github.com/Zaretto/f-14b/blob/release/1.12/Nasal/instruments.nas#L551-L670)
```nasal
var instruments_exec = {
    ...
    update : func(notification) {
        if ( !math.mod(notifications.frameNotification.FrameCount,2)){
            tacan_update();
            f14_hud.update_hud();
            g_min_max();
            f14_chronograph.update_chrono();
        } else {
            awg_9.hud.hud_nearest_tgt();
            instruments_data_export();
            afcs_filters();
        }
    },
};
```

A concrete example: the G-meter. Nasal reads the current G, tracks min/max, computes a moving average, and writes to instrumentation properties:

[Nasal/instruments.nas Lines 314-336](https://github.com/Zaretto/f-14b/blob/release/1.12/Nasal/instruments.nas#L314-L336)
```nasal
var g_min_max = func {
    var curr = f14.currentG;
    var max = g_max.getValue();
    if ( curr >= max ) {
        g_max.setDoubleValue(curr);
    }
    # ... moving average written to g-meter/g-max-mooving-average
}
```

The fuel system is another good example - `fuel-system.nas` reads tank levels, computes totals per group, and writes to gauge properties every 0.3 sec ([Nasal/fuel-system.nas line 608](https://github.com/Zaretto/f-14b/blob/release/1.12/Nasal/fuel-system.nas#L608)).

Some systems straddle both - the CADC (Central Air Data Computer) is implemented as a JSBSim system with Mach/AOA-scheduled flap and slat tables ([Systems/f-14b-cadc.xml](https://github.com/Zaretto/f-14b/blob/release/1.12/Systems/f-14b-cadc.xml)), while the sweep computer logic is in Nasal ([Nasal/sweep_computer.nas](https://github.com/Zaretto/f-14b/blob/release/1.12/Nasal/sweep_computer.nas)).

## 3. Animation Bindings - XML Connecting Properties to Geometry

This is where properties become visible movement. In the model XML files, `<animation>` elements bind a property path to a transformation on a named 3D object from the .ac file.

Rudder - reads `surface-positions/rudder-pos-norm` (written by JSBSim FCS), rotates the rudder object +/-30 degrees about its hinge axis:

[Models/f-14b.xml Lines 1145-1160](https://github.com/Zaretto/f-14b/blob/release/1.12/Models/f-14b.xml#L1145-L1160)
```xml
<animation>
    <type>rotate</type>
    <object-name>RudderL</object-name>
    <property>surface-positions/rudder-pos-norm</property>
    <factor>-30</factor>
    <center>
        <x-m>7.531</x-m> <y-m>-1.496</y-m> <z-m>-0.481</z-m>
    </center>
    <axis> <x>-1.328</x> <y>0.212</y> <z>-2.388</z> </axis>
</animation>
```

Airspeed indicator - reads `instrumentation/airspeed-indicator/indicated-speed-kt` (written by FlightGear's C++ instrument subsystem), but only when the AC essential bus is powered. Uses an interpolation table mapping knots to needle rotation degrees:

[Models/Cockpit/all-cockpit.xml Lines 1428-1474](https://github.com/Zaretto/f-14b/blob/release/1.12/Models/Cockpit/all-cockpit.xml#L1428-L1474)
```xml
<animation>
    <type>rotate</type>
    <object-name>asi-inner-pointer-dial</object-name>
    <property>instrumentation/airspeed-indicator/indicated-speed-kt</property>
    <condition>
        <greater-than>
            <property>fdm/jsbsim/systems/electrics/ac-essential-bus1</property>
            <value>0</value>
        </greater-than>
    </condition>
    <interpolation>
        <entry><ind>0</ind><dep>0</dep></entry>
        <entry><ind>80</ind><dep>28</dep></entry>
        <entry><ind>200</ind><dep>134</dep></entry>
        <entry><ind>500</ind><dep>279</dep></entry>
    </interpolation>
</animation>
```

Landing gear uses both rotate and translate animations with interpolation to sequence the retraction geometry, driven by `gear/gear[0]/position-norm` (C++ gear subsystem output).

Clickable controls use `<type>pick</type>` animations with Nasal bindings that fire when clicked, e.g. the radar range buttons in the cockpit call directly into the AWG-9 Nasal module:

[Models/Cockpit/all-cockpit.xml Lines 39-47](https://github.com/Zaretto/f-14b/blob/release/1.12/Models/Cockpit/all-cockpit.xml#L39-L47)
```xml
<animation>
    <type>pick</type>
    <object-name>rng-5-button</object-name>
    <action>
        <binding>
            <command>nasal</command>
            <script>awg_9.RangeRadar2.setValue(5);</script>
        </binding>
    </action>
</animation>
```

## 4. The C++ Part (SimGear/FlightGear)

The C++ you'd hit with breakpoints is essentially infrastructure:
- **SGAnimation** (SimGear) - reads bound properties each frame and applies transforms to OSG scene graph nodes. `SGRotateAnimation`, `SGTranslateAnimation`, `SGTextAnimation` etc.
- **JSBSim** (FDM) - the C++ engine that evaluates all those XML tables, filters, summers, kinematics
- **Property tree** (`SGPropertyNode`) - the in-memory data store everything reads and writes
- **Input subsystem** - maps joystick/keyboard to `fcs/*-cmd-norm` properties

Aircraft authors don't write C++. The creative work is XML (FDM + animations) and Nasal (systems logic). C++ is the execution engine underneath.

## 5. The Full Chain for One Thing

Pick the rudder. End to end:

1. **Pilot pushes pedal** → FlightGear input system writes `fcs/rudder-cmd-norm` (C++)
2. **JSBSim FCS** processes through yaw damper, SAS, hydraulic check, rate limiter → writes `surface-positions/rudder-pos-norm` (C++ executing XML)
3. **JSBSim aero model** uses deflection in yaw moment coefficient tables from AFWAL windtunnel data → computes forces (C++ executing XML)
4. **SGRotateAnimation** reads `surface-positions/rudder-pos-norm`, rotates the `RudderL`/`RudderR` objects in the OSG scene graph (C++)

---

*Richard Harrison (rjh@zaretto.com) — [zaretto.com/f-14](http://zaretto.com/f-14)*
