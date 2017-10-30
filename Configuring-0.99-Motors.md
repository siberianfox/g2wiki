_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

# Motor Groups
This page documents motor settings. Each motor object ("group") has a collection of parameters for that motor. There are 6 motor groups, numbered 1,2,3,4,5,6. These are labeled on the v9 board, which breaks out motor1 - motor4. Other platforms may make more or fewer motors available, e.g. the Due has outputs for all 6 motors, which are labeled [here](Arduino-DUE-Pinout-for-g2core)

- Motor examples use Motor 1, but any motor active for your platform is OK

## Summary

Setting | Description | Notes
--------|-------------|-----------------------------
________________|_________________|
[{1:{ma:...}}](#1ma-map-motor-to-axis) | **M**ap to **A**xis | Configure axis to which this motor is connected (for Cartesian machines) E.g. {1ma:0}, {2ma:1}, {3ma:2}, {4ma:3} to map motors 1-4 to X,Y,Z,A, respectively
[{1:{sa:...}}](#1sa-step-angle) | **S**tep **A**ngle | Motor parameter indicating the angle traveled per whole step. Typical setting is {1sa:1.8} for 1.8 degrees per step (200 steps per revolution)
[{1:{tr:..}}](#1tr-travel-per-revolution) | **T**ravel per **R**evolution | How far the mapped axis moves per motor revolution. E.g. {1tr:2.54} (millimeters) for a 10 TPI screw axis
[{1:{mi:...}}](#1mi-microsteps) | **MI**crosteps | Microsteps per whole step. G2 uses 1,2,4,8,16,32. Other values are accepted but warned
[{1:{su:...}}](#1su-steps-per-unit) | **S**teps per **U**nit | Direct input and reading of steps per unit - about 7 digits decimal accuracy
[{1:{po:..}}](#1po-polarity) | **PO**larity | Set polarity for proper movement of the axis. 0=clockwise rotation, 1=counterclockwise - although these are dependent on your motor wiring, and axis movement is dependent on the mechanical system.
[{1:{pm:...}}](#1pm-power-management) | **P**ower **M**ode | 0=motor disabled, 1=motor always on, 2=motor on when in cycle, 3=motor on only when moving
[{1:{pl:...}}](#1pl-power-level) | **P**ower **L**evel | 0.000=no power to steppers, 1.000=max power to steppers

_Note: Setting power level to every high values will not necessarily get more power to the motors. At some point you may  saturate the coils, get worse motor performance, draw more current, and risk burning out the motors by overheating._

## Motor Settings
Motor travel settings can be done one of two ways. You can enter the steps per unit `{su:...}`, which sets the number of steps the code will emit to travel one complete unit - millimeter, inch, or degree. the SU value is accurate to roughly 7 decimal places, with 5 places available to the right of the decimal point.

Or you can enter the parameters that compute the steps per unit, which are Step Angle: `{1:{sa:...}}`, Travel per Revolution: `{1:{tr:...}}` and Microsteps: `{1:{mi:...}}`. For example, if you want run motor #1 with a 200 step per revolution motor, a 3mm GT2 timing belt with a 20 tooth pulley, and 8 microsteps you enter: 

- Step angle: `{1:{sa:1.8}}` or `{1sa:1.8}`
- Travel per revolution: `{1tr:20}`  
- Microsteps: `{1mi:8}`

...and it will compute and set the SU value. If the SU is set directly the TR value is changed to agree with the SU value. The values entered for step angle and microsteps are not changed.

### {1:{ma:...}} Map Motor to Axis
Axes must be input as numbers, with X=0, Y=1, Z=2, A=3, B=4 and C=5. As you might expect, mapping motor 1 to X will cause X movement to drive motor 1. The example below is a way to run a dual-Y gantry such as a 4 motor Shapeoko2 setup. Movement in Y will drive both motor2 and motor4. 

<pre>
{1:{ma:0}}  Map motor 1 to X axis
{2:{ma:1}}  Map motor 2 to Y1 axis
{3:{ma:1}}  Map motor 3 to Y2 axis
{4:{ma:2}}  Map motor 4 to Z axis
</pre> 

### {1:{sa:...}} Step Angle
This is a decimal number which is often 1.8 degrees per step, but should reflect the motor in use. You might also find 0.9, 3.6, 7.5 or other values. You can usually read this off the motor label. If a motor is indicated in steps per revolution just divide 360 by that number. A 200 step-per-rev motor is 1.8 degrees, a 400 step-per-rev motor has 0.9 degrees per step.

<pre>
{1:{sa:1.8}} This is a typical value for many motors 
</pre> 

### {1:{tr:...}} Travel per Revolution
TR needs to be set to the distance the mapped axis will move for one revolution of the motor. - e.g. if motor 1 is mapped to the X axis, then 1tr applies to the Xaxis. If the machine is in mm mode (G21) the TR value for XYZ axes should be entered in mm. If in inches mode (G20) XYZ should be entered in inches. ABC axes are always entered in degrees.

<pre>
{1:{tr:2.54}}  Sets motor 1 to a 10 TPI travel from millimeters (2.54 mm per revolution)
</pre>

For XYZ the travel-per-revolution value is usually the result of the lead screw pitch or pulley circumference.
* A 10 thread-per-inch (TPI) lead screw moves 0.100" per revolution. TR in inches would be 0.100, or 2.54 in mm mode. 
* A 0.500" radius pulley will travel 3.14159" per revolution, absent any other gearing. A typical value for a Shapeoko2 or Reprap belt driven machine is on the order of 36.540 mm per revolution. Don't take this as exact - you will need to do your own calibration on your machine to get this setting exact.

For ABC the travel-per-revolution value is entered in degrees. This value will be 360 degrees for an axis that is not geared down - one revolution = 360 degrees. The value for a geared rotary axis is 360 divided by the gear ratio. For example, a motor-driven rotary table with 4 degrees of table movement per handle rotation has a gear ratio of 90:1. The Travel per Revolution value should be set to 4. 

Note that the travel-per-revolution is independent of the radius setting in the rotary axis settings. Set TR first to reflect the gearing, then set any Radius values if that is needed.  

Note that Travel per Revolution is a motor parameter, not an axis parameter as one might think. Consider the case of a dual Y gantry with lead screws of different pitch (how weird). The travel per revolution would be different for each motor. 

### {1:{mi..}} Microsteps
G2core microsteps are set in firmware, not as hardware jumpers as on some other systems. The following microstep values are supported: 

<pre>
{1:{mi:1}}   no microsteps (whole steps)
{1:{mi:2}}   half stepping
{1:{mi:4}}   quarter stepping
{1:{mi:8}}   eighth stepping
{1:{mi:16}}  sixteenth stepping
{1:{mi:32}}  thirty second stepping
</pre>

G2core can also drive external stepper drivers using the breakout headers. Some drivers use other values than the above, so any value is accepted. Values other than those above are warned as non-standard.

_Note about Microsteps: It is a misconception that higher microstep values are better - beyond a certain point they degrade motor performance. In a typical setup the total power delivered to the motor (and hence torque) will go down as you increase the microsteps, especially at higher speeds. Also, using microsteps to set the finest machine resolution is source of error as the shaft angle isn't necessarily going to be at the theoretical point. Don't just assume that 1/32 microstepping is the right setting for your application. Try out different settings to balance smoothness and power._

### {1:{su:...}} Steps per Unit
This is a floating point number that sets the number of steps the code will emit to travel one complete unit - millimeter, inch, or degree. the SU value is accurate to roughly 7 decimal places, with 5 places available to the right of the decimal point.

### {1:{po:...}} Polarity
Set to one of the following: 

<pre>
{1:{po:0}}  Normal motor polarity
{1:{po:1}}  Invert motor polarity
</pre>

Polarity sets which direction the motor will turn when presented with positive and negative Gcode coordinates. It's affected by how you wired the motors and by mechanical factors. Set polarity so the indicated axis travels in the correct orientation for your machine. 

Travel in X and Y is dependent on the conventions for your particular machine and CAD setup. Typically X is left/right movement, and Y is towards and away from you, but people often set up the machine to agree with the visualization their CAD program provides, and can depend on where you stand when operating the machine. Typically +X moves to the right, -X to the left, +Y away from you, and -Y towards you. Z is by convention the cutting axis, which is the vertical axis on a typical milling machine. +Z should move up, and -Z should move down, into the work.

### {1:{pm:...}} Power Management
Power management is used to keep the steppers on when you need them and turn them off when you don't. See [Power Management](Power-Management) page.

### {1:{pl:...}} Power Level
Power level is used to set the stepper driver current. `0` is off, `1.000` is maximum, which is about 2.5 amps for the DRV8825 drivers used on most G2 boards.

Set the current carefully, since the value can be too high or too low. Is short: a higher power level does *not* necessarily mean a higher torque. Underdriving the motor can yield skipped steps and too low of torque. Overdriving the motor typically yields motor performance that's actually worse than a more moderate power setting. When overdriving the motor "runs rough" (often caused by it skipping over microsteps to full-step positions), heats up excessively, and may even cause damage to the motors over time. If there is too much current the drivers may also heat up until they go into thermal shutdown to protect themselves.

# Config Files to enable more motors

4 motors are enabled by default on the default build target: gShield on ArduinoDue Board.
To enable more motors, edit the following: board_stepper.h (uncomment motor_5) board_stepper.cpp (uncomment motor_5) hardware.h (#define MOTORS 5) settings_shapeoko2.h (define the list of M5_ settings)