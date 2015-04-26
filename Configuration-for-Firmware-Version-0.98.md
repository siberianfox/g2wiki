**The settings on this page are for firmware version 0.98**
The version number can be found as the fv variable in the startup JSON message, or by entering {fv:n} or $fv. Version 0.98 starts with g2 build 082.06.

###Conventions Used on this Page
- Examples show relaxed JSON mode (e.g. `{fv:n}`). Strict JSON is also accepted in all cases (e.g. `{"fv":null}`)
- CMD means some command - aka the "name" of the name/value pair. CMDs are case insensitive.
- Underscore "_" means some numeric value
- "abcd" means some string value
- Motor examples use Motor 1, but any motor active for your platform is OK
- Axis examples use X axis, but any axis is OK unless otherwise noted
- All examples are in millimeter units

**A note about units:**
- All data entry and display occurs in the prevailing units set in the Gcode mode. Set G20 for inches, G21 for mm. Query {unit:n} or $unit to find out where you are (G20=0, G21=1)
- Units entered in one system are correctly preserved and do not need to be re-entered or adjusted if the Gcode units mode changes.
- The Gcode default units setting `{gun:_}` sets the units the system "comes up in" during power-on or reset. So changing GUN will not change your units until you reset. Note: PROGRAM END (M2, M30) does not change the units. 

# Summary / Cheat Sheet
### JSON Cheat Sheet
All configuration commands are available using [JSON mode](JSON-Operation), which is the preferred access method if you are writing a UI or controller. Most commands are also available in [Text Mode](Text-Mode-Operation). The few commands that are available from only one or the other are noted, as are any commands the behave differently depending on the mode. The rough equivalence is:
<pre>
{"CMD":n} == $CMD            Read value for command "CMD" in strict JSON mode
{CMD} == $CMD                Read value in relaxed JSON mode
{"CMD":123.4} == $CMD=123.4  Set value to 123.4 in strict JSON mode
{CMD:123.4} == $CMD=123.4    Set value in relaxed JSON mode
{xvm:50000}                  Set X max velocity to 50000 as an example
</pre>
You can also use JSON to read an entire object - examples for motor and axis:
<pre>
{1:n}
{"r":{"1":{"ma":0,"sa":1.800,"tr":40.0000,"mi":8,"po":0,"pm":2,"pl":0.375}},"f":[1,0,5]}

{x:n}
{"r":{"x":{"am":1,"vm":50000,"fr":50000,"tn":0.000,"tm":420.000,"jm":10000,"jh":20000,"jd":0.1000,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":20.000,"zb":3.000}},"f":[1,0,5]}
</pre>
The footer is an array of 3 elements:
- (1) Footer revision
- (2) [Status code](Status-Codes) - Short version: status=0 is OK, everything else is an exception.
- (3) RX buffer info (explained later)
- Note: The checksum that used to be in the footer has been removed as no-one was using it.

##Motor Groups
Settings specific to a given motor. There are 6 motor groups, numbered 1,2,3,4,5,6. These are labeled on the v9 board, which breaks out motor1 - motor4. Other platforms may make more or fewer motors available, e.g. the Due has outputs for all 6 motors, which are labeled [here](Arduino-DUE-Pinout-for-tinyG2).<br><br>

	Setting | Description | Notes
	--------|-------------|-----------------------------
	[{1ma:_}](#1ma---map-motor-to-axis) | Map to Axis | Configure axis to which this motor is connected (for Cartesian machines) E.g. {1ma:0}, {2ma:1}, {3ma:2}, {4ma:3} to map motors 1-4 to X,Y,Z,A, respectively
	[{1sa:_}](#1sa---step-angle-for-the-motor) | Step Angle | Motor parameter indicating the angle traveled per whole step. Typical setting is {1sa:1.8} for 1.8 degrees per step (200 steps per revolution)
	[{1tr:_}](#1tr---travel-per-revolution) | Travel_per_Rev | How far the mapped axis moves per motor revolution. E.g. {1tr:2.54} (millimeters) for a 10 TPI screw axis
	[{1mi:_}](#1mi---microsteps) | MIcrosteps | Microsteps per whole step. G2 uses 1,2,4,8,16,32. Other values are accepted but warned
	[{1po:_}](#1po---polarity) | POlarity | Set polarity for proper movement of the axis. 0=clockwise rotation, 1=counterclockwise - although these are dependent on your motor wiring, and axis movement is dependent on the mechanical system.
	[{1pm:_}](Power-Management) | Power Mode | 0=motor disabled, 1=motor always on, 2=motor on when in cycle, 3=motor on only when moving
	[{1pl:_}](#1pl---power-level) | Power Level | 0.000=no power to steppers, 1.000=max power to steppers

## Axis Groups
Settings specific to a given axis. There are 6 axis groups, X,Y,Z linear axes, and A,B,C rotary axes. Not all axes have all parameters.

	Setting | Description | Notes
	--------|-------------|-------
	[{xam:_}](#xam---axis-mode) | Axis mode | Normally {xam:1} "normal". See details for setting. 
	[{xvm:_}](#xvm---velocity-maximum) | Velocity maximum | Max velocity for axis, aka "traverse rate" or "seek" 
	[{xfr:_}](#xfr---feed-rate-maximum) | Feed_rate_maximum | Sets maximum feed rate for that axis. Does NOT set the Gcode F word
	[{xtn:_}](#xtn-xtm---travel-minimum-travel-maximum) | Travel minimum | Minimum travel in absolute coordinates. Used by homing and soft limits 
	[{xtm:_}](#xtn-xtm---travel-minimum-travel-maximum) | Travel maximum | Maximum travel in absolute coordinates. Used by homing and soft limits 
	[{xjm:_}](#xjm---jerk-maximum) | Jerk maximum | Main parameter for acceleration management
	[{xjh:_}](#xjh---jerk-high) | Jerk High | Jerk used during homing operations
	[{xjd:_}](#xjd---junction-deviation) | Junction deviation | Sets the imaginary radius For cornering. Larger values yield faster cornering but more corner jerk.
	[{ara:_}](#ara---radius-value) | Radius setting | Artificial radius to convert linear values to degrees. ABC axes only.
	[{xhi:_}](#homing-settings) | Homing Input | Switch (input) to use for homing this axis
	[{xhd:_}](#homing-settings) | Homing Direction | 0=search-towards-negative, 1=search-torwards-positive
	[{xsv:_}](#homing-settings) | Search velocity | Homing speed during search phase (drive to switch)
	[{xlv:_}](#homing-settings) | Latch velocity | Homing speed during latch phase (drive off switch)
	[{xlb:_}](#homing-settings) | Latch backoff | Maximum distance to back off switch during latch phase (drive off switch)
	[{xzb:_}](#homing-settings) | Zero backoff | Offset from switch for zero in absolute coordinates

##PWM Group (Pulse Width Modulation)
There is currently only one PWM channel (p1), but the configs are structured for multiple PWM groups. The PWM channel is set up to act as a remote control Electronic Speed Controller (ESC), but can be used for other PWM functions using these settings. 

	Setting | Description | Notes
	--------|-------------|-------
	{p1frq:_} | Frequency | in Hz, e.g. 100
	{p1csl:_} | Clockwise speed low | In RPM - arbitrary units unless you calibrate it, e.g. 1000
	{p1csh:_} | Clockwise speed high | In RPM
	{p1cpl:_} | Clockwise phase low | 0.000 to 1.000, e.g. 0.125 for 12.5% phase angle
	{p1cph:_} | Clockwise_phase_high | 0.000 to 1.000
	{p1wsl:_} | CCW speed low | In RPM 
	{p1wsh:_} | CCW speed high | In RPM
	{p1wpl:_} | CCW phase low | 0.000 to 1.000
	{p1wph:_} | CCW phase high | 0.000 to 1.000
	{p1pof:_} | Phase off | 0.000 to 1.000 used to set OFF phase for PWM devices that are not off at 0 phase

## System Group
The system group contains the following global machine and communication settings. The system group can be listed by requesting {sys:n} or $sys

**Identification Settings**
These are reported on the startup strings and should be included in any support discussions.

	Setting | Description | Notes
	--------|-------------|-------
	[{fb:n}](#fb---firmware-build-number) | Firmware Build | Read-only value, e.g. 82.06
	[{fv:n}](#fv---firmware-version) | Firmware Version | Read-only value, e.g. 0.98
	[{hp:n}](#hp---hardware-platform) | Hardware_Platform | Read-only value, 1=Xmega, 2=Due, 3=v9(ARM)
	[{id:n}](#id---unique-board-identifier) | board ID | Each board has a read-only unique ID
	[{hv:_}](#hv---hardware-version) | Hardware Version | **V8 only** Set to 6 for v6 and earlier boards, 7 or 8 for v7 and v8 boards. Defaults to 8. 

**Global System Settings**

	Setting | Description | Notes
	--------|-------------|-------
	[{ja:_}](#ja---junction-acceleration) | Junction Acceleration | Global cornering acceleration value
	[{ct:_}](#ct---chordal-tolerance) | Chordal Tolerance | Sets precision of arc drawing. Trades off precision for max arc draw rate 
	[{mt:_}](#mt---motor-power-timeout) | Motor_disable_Timeout | Number of seconds before motor power is automatically released. Maximum value is 40 million.


**Communications Settings**
Set communications speeds and modes. 

	Setting | Description | Notes
	--------|-------------|-------
	[{ej:_}](#ej---enable-json-mode-on-power-up) | Enable JSON mode | 0=text mode, 1=JSON mode
	[{jv:_}](#jv---set-json-verbosity) | JSON verbosity | 0=silent ... 5=verbose
	[{js:_}](#js---set-json-syntax) | JSON syntax | 0=relaxed, 1=strict
	[{tv:_}](#tv---set-text-mode-verbosity) | Text mode verbosity | 0=silent, 1=verbose
	[{qv:_}](#qv---queue-report-verbosity) | Queue report verbosity | 0=off, 1=filtered, 2=verbose
	[{sv:_}](#sv---status-report-verbosity) | Status_report_Verbosity | 0=off, 1=filtered, 2=verbose
	[{si:_}](#si---status-interval) | Status report interval | in milliseconds (100 ms minimum interval)

V8-Only Communications Settings (not available on g2)

	Setting | Description | Notes
	--------|-------------|-------
	[{ec:_}](#ec---expand-lf-to-crlf-on-tx-data) | Enable CR on TX | 0=send LF line termination on TX, 1= send both LF and CR termination
	[{ee:_}](#ee---enable-character-echo) | Enable_character_Echo | 0=off, 1=enabled
	[{ex:_}](#ex---enable-flow-control) | Enable flow control | 0=off, 1=XON/XOFF enabled, 2=RTS/CTS enabled
	[{baud:_}](#baud---set-usb-baud-rate) | Baud rate | 1=9600, 2=19200, 3=38400, 4=57600, 5=115200, 6=230400 -- 115200 is default


**Gcode Initialization Defaults**
Gcode settings loaded on power up and reset. Changing these does NOT change the current Gcode state, only the initialization settings. 

	Setting | Description | Notes
	--------|-------------|-------
	[{gpl:_}](#gpl---gcode-default-plane-selection) | PLane selection | 0=XY plane (G17), 1=XZ plane (G18), 2=YZ plane (G1)
	[{gun:_}](#gun---gcode-default-units) | UNits mode | 0=inches mode (G20), 1=mm mode (G21) 
	[{gco:_}](#gco---gcode-default-coordinate-system) | COordinate system | 1=G54, 2=G55, 3=G56, 4=G57, 5=G58, 6=G59
	[{gpa:_}](#gpa---gcode-default-path-control) | PAth_control_mode | 0=Exact path mode (G61), 1=Exact stop mode (G61.1), 2=Continuous mode (G64)
	[{gdi:_}](#gdi---gcode-distance-mode) | Distance mode | 0=Absolute mode (G90), 1=Incremental mode (G91)

##Commands and Reports
These $configs invoke reports and functions

	Command | Description | Notes
	--------|-------------|-------
	[{sr:n}](#sr---status-report) | get Status Report | SR also sets status report format in JSON mode
	[{qr:n}](#qr---queue-report) | get Queue Report | 
	[{qf:_}](#qf---queue-flush) | flush_planner_queue | Used with '!' feedhold for jogging, probes and other sequences. Usage: {qf:1}
	[{md:n}](Power-Management) | Disable motors | Unpower all motors
	[{me:_}](Power-Management) | Energize motors | Energize all motors with timeout in seconds 
	[{test:_}](#test---run-self-test) | Invoke self tests | $test=n for test number; $test returns help screen in text mode
	[{defa:1}](#defa---reset-default-profile-settings) | reset to DEFAults | Reset to compiled default settings
	[{flash:1}] | Enter_FLASH_loader | WILL ERASE AND REFLASH BOARD. BE CAREFUL WITH THIS
        ^x | Reset tinyG | cntl+x restarts tinyG same as hardware rest button
	[{help:_}] | Show help screen | Show system help screen; $h also works

Note: Status report parameters is settable in JSON only - see JSON mode for details
<br>
# Settings Details
## Motor Settings
_Note: In TinyG the motor travel settings are independent of each other. You don't need to put them into an equation - the board does that for you. For example, if you want run motor #1 with a 200 step per revolution motor, a 3mm GT2 timing belt with a 20 tooth pulley, and 8 microsteps you simply enter:_
* $1sa=1.8
* $1tr=60   (which is 3 * 20)
* $1mi=8

### 1ma - Map Motor to Axis
Axes must be input as numbers, with X=0, Y=1, Z=2, A=3, B=4 and C=5. As you might expect, mapping motor 1 to X will cause X movement to drive motor 1. The example below is a way to run a dual-Y gantry such as a 4 motor Shapeoko2 setup. Movement in Y will drive both motor2 and motor4. 

<pre>
{1ma:0}    Map motor 1 to X axis
{2ma:1}    Map motor 2 to Y1 axis
{3ma:1}    Map motor 3 to Y2 axis
{4ma:2}    Map motor 4 to Z axis
</pre> 

### 1sa - Step Angle
This is a decimal number which is often 1.8 degrees per step, but should reflect the motor in use. You might also find 0.9, 3.6, 7.5 or other values. You can usually read this off the motor label. If a motor is indicated in steps per revolution just divide 360 by that number. A 200 step-per-rev motor is 1.8 degrees, a 400 step-per-rev motor has 0.9 degrees per step.

<pre>
{1sa:1.8}  This is a typical value for many motors 
</pre> 

### 1tr - Travel per Revolution
TR needs to be set to the distance the mapped axis will move for one revolution of the motor. - e.g. if motor 1 is mapped to the X axis, then 1tr applies to the Xaxis. If the machine is in mm mode (G21) the TR value for XYZ axes should be entered in mm. If in inches mode (G20) XYZ should be entered in inches. ABC axes are always entered in degrees.

<pre>
{1tr:2.54}   Sets motor 1 to a 10 TPI travel from millimeters (2.54 mm per revolution)
</pre>

For XYZ the travel-per-revolution value is usually the result of the lead screw pitch or pulley circumference.
* A 10 thread-per-inch (TPI) leadscrew moves 0.100" per revolution. TR in inches would be 0.100, or 2.54 in mm mode. 
* A 0.500" radius pulley will travel 3.14159" per revolution, absent any other gearing. A typical value for a Shapeoko2 or Reprap belt driven machine is on the order of 36.540 mm per revolution. Don't take this as exact - you will need to do your own calibration on your machine to get this setting exact.

For ABC the travel-per-revolution value is entered in degrees. This value will be 360 degrees for an axis that is not geared down - one revolution = 360 degrees. The value for a geared rotary axis is 360 divided by the gear ratio. For example, a motor-driven rotary table with 4 degrees of table movement per handle rotation has a gear ratio of 90:1. The Travel per Revolution value should be set to 4. 

Note that the travel-per-revolution is independent of the radius setting in the rotary axis settings. Set TR first to reflect the gearing, then set any Radius values if that is needed.  

Note that Travel per Revolution is a motor parameter, not an axis parameter as one might think. Consider the case of a dual Y gantry with lead screws of different pitch (how weird). The travel per revolution would be different for each motor. 

### 1mi - Microsteps
G2 microsteps are set in firmware, not as hardware jumpers as on some other systems. The following microstep values are supported: 

<pre>
{1mi:1}   no microsteps (whole steps)
{1mi:2}   half stepping
{1mi:4}   quarter stepping
{1mi:8}   eighth stepping
{1mi:16}  sixteenth stepping
{1mi:32}  thirty second stepping
</pre>

G2 can also drive external stepper drivers using the breakout headers. Some drivers use other values than the above, so any value is accepted. Values other than those above are warned as non-standard.

_Note about Microsteps: It is a misconception that higher microstep values are better - beyond a certain point they degrade motor performance. In a typical setup the total power delivered to the motor (and hence torque) will go down as you increase the microsteps, especially at higher speeds. Also, using microsteps to set the finest machine resolution is source of error as the shaft angle isn't necessarily going to be at the theoretical point. Don't just assume that 1/32 microstepping is the right setting for your application. Try out different settings to balance smoothness and power._

### 1po - Polarity
Set to one of the following: 

<pre>
{1po:0}   Normal motor polarity
{1po:1}   Invert motor polarity
</pre>

Polarity sets which direction the motor will turn when presented with positive and negative Gcode coordinates. It's affected by how you wired the motors and by mechanical factors. Set polarity so the indicated axis travels in the correct orientation for your machine. 

Travel in X and Y is dependent on the conventions for your particular machine and CAD setup. Typically X is left/right movement, and Y is towards and away from you, but people often set up the machine to agree with the visualization their CAD program provides, and can depend on where you stand when operating the machine. Typically +X moves to the right, -X to the left, +Y away from you, and -Y towards you. Z is by convention the cutting axis, which is the vertical axis on a typical milling machine. +Z should move up, and -Z should move down, into the work.

### 1pm - Power Management
Power management is used to keep the steppers on when you need them and turn them off when you don't. See [Power Management](Power-Management) page.

### 1pl - Power Level
Power level is used to set the stepper driver current. 0 is off, 1.000 is maximum, which is about 2.5 amps for the DRV8825 drivers used on most G2 boards. Set the current carefully, as overdriving the motor typically yields motor performance that's actually worse than a more moderate power setting. The motor "runs rough", heats up excessively and may even burn out. If there is too much current the drivers may also go into thermal shutdown to protect themselves.

## Axis Settings

### xam - Axis Mode
Sets the function of the axis.

- {xam:0}  Disable. All input to that axis will be ignored and the axis will not move. 
- {xam:1}  Standard. Linear axes move in length units. Rotary axes move in degrees. 
- {xam:2}  Inhibited. Axis values are taken into account when planning moves, but the axis will not move. Use this to perform a Z kill or to do a compute-only run.
- {xam:3}  Radius mode. (Rotary axes only) In radius mode gcode values are interpreted as linear units; either inches or mm depending on the prevailing G20/G21 setting. The conversion of linear units to degrees is accomplished using the radius setting for that axis. See $aRA for details. 

### xvm - Velocity Maximum
(aka traverse rate or seek rate). Sets the maximum velocity the axis will move during a G0 traverse move. This is set in length units per minute for linear axes, degrees per minute for rotary axes. 

Note that the max velocity is *per-axis*. Diagonal / multi-axis traverses will actually occur at the fastest speed the combined set of axes and the geometry will allow, and may be faster than the individual axis max velocities. For example, max velocity for X and Y are set to 1000 mm/min. For a 45 degree traverse in X and Y the toolhead would travel at 1414.21 mm/min. 

<pre>
{xvm:1200}   set X maximum velocity (G0) to 1200 mm/min
{avm:36000}  set A to 100 revolutions per minute (360 * 100)
{zvm:30.0}   set Z to 30 inches per minute - in G20 / inches mode
</pre>
 
### xfr - Feed Rate Maximum
Sets the maximum velocity the axis will move during a feed in a G1, G2, or G3 move. This works similarly to maximum velocity, but instead of actually setting the speed, it only serves to establish a "do not exceed" for Gcode F words. Put another way, the maximum feed rate setting is NOT used to set the Gcode's F value; it is only a maximum that may be used to limit the F value provided in a gcode file.

Axis feed rates should be equal to or less than the maximum velocity. See [G2 Tuning](G2-Tuning) for more details. 

<pre>
{xfr:1000}  set X max feed rate to 1000 mm/min
</pre> 

### xtn, xtm - Travel Minimum, Travel Maximum
Defines the minimum and maximum extent of travel in that axis. This is used during homing and for soft limits. See [Homing](Homing-Operation) for homing details

Both values can be positive or negative, but maximum must be greater than minimum or equal to minimum. If minimum and maximum are equal the axis is treated as an infinite axis (i.e. no limits). This is useful for rotary axes - for example:
<pre>
{xtn:-1}
{xtm:-1}
or
{xtn:0}
{xtm:0}
</pre> 

### xjm - Jerk Maximum
Sets the maximum jerk value for that axis. Jerk is settable independently for each axis to support machines with different dynamics per axis - such as Shapeoko2 with belts for X and Y, screws for Z, Probotix with 5 pitch X and Y screws and 12 pitch Z screws, and any machine with both linear and rotary axes.

Jerk is in units per minutes^3, so the numbers are quite large (but see note below). Some common values are shown in *millimeters* in the examples below. 

To manage these large numbers better the complete number can be entered, or the number divided by a million. Any number less than 1M will be automatically multiplied by 1M internally. Jerk values are displayed in divide-by-million form.

<pre>
{xjm:50000000}  Set X jerk to 50 million MM per min^3. This is a good value for a moderate speed machine
{xjm:50}        Same as above
{xjm:5000}      X jerk for Shapeoko. Yes, that's 5 billion
</pre> 

The jerk term in mm is measured in mm/min^3. In inches mode it's units are inches/min^3. So the conversion from mm to inches is 1/(25.4). The same values as above are shown in inches are: 
<pre>
50,000,000 mm/min^3      is 1,968,504 in/min^3 2,000,000 would suffice
25,000,000 mm/min^3      is 984,251 in/min^3 1,000,000 would suffice
5,000,000,000 mm/min^3   is 196,850,400 in/min^3 200,000,000 would suffice
</pre> 

### xjh - Jerk High
Sets the jerk value used for homing to stop movement when switches are hit or released. You generally want this value to be larger than the xJM value, as this determines how fast the axis will stop once it hits the switch. You generally want this as fast as you can get it without losing steps on the accelerations.

### xjd - Junction Deviation
This one is somewhat complicated. Junction deviation - in combination with Junction Acceleration ($JA) from the system group - sets the velocity reduction used during cornering through the junction of two lines. The reduction is based on controlling the centripetal acceleration through the junction to the value set in JA with the junction deviation being the "tightness" of the controlling cornering circle. An explanation of what's happening here can be found on [Sonny Jeon's blog: Improving grbl cornering algorithm] (http://onehossshay.wordpress.com/2011/09/24/improving_grbl_cornering_algorithm/ onehossshay.wordpress.com/2011/09/24/improving_grbl_cornering_algorithm/). 

It's important to realize that the tool head does not actually follow the controlling circle - the circle is just used to set the speed of the tool through the defined path. In other words, the tool does go through the sharp corner, just not as fast. This is a Gcode G61 - Exact Path Mode operation, not a Gcode G64 - Continuous Path Mode (aka corner rounding, or splining) operation. 

While JA is set globally and applies to all axes, JD is set per axis and can vary depending on the characteristics of the axis. An axis that moves more slowly should have a JD that is less than an axis that can move more quickly, as the larger the JD the faster the machine will move through the junction (i.e. a bigger controlling circle). The following example has some representative values for a Probotix Fireball V90 machine. The V90 has 5 TPI X and Y screws, and 12 TPI Z. All values in MM. 

<pre>
{ja:200000}  Units are mm/min^2
{xjd:0.05}   Units are mm
{yjd:0.05}
{zjd:0.02}   Setting Z to a smaller value means that moves with a change in the Z component will move proportionately slower depending on the contribution in Z. 
</pre>

### ara - Radius value
The radius value is used by rotational axes only (A, B and C) to convert linear units to degrees when in radius mode. 

For example; if the A radius is set to 10 mm it means that a value of 62.8318531 mm will make the A axis travel one full revolution - as 62.383... is the circumference of the circle of radius R ( 2*PI*R, or 10 * 2 * 3.14159...) (Assuming nTR = 360 -- see note below). Receiving the gcode block `G0 A62.83` will turn the A axis one full revolution (360 degrees) from a starting position of 0. All internal computations and settings are still in degrees - it's just that gcode units received for the axis are converted to degrees using the specified radius. 

Note that the Travel per Revolution value (1tr) is used but unaffected in radius mode. The degrees per revolution still applies, it's just that the degrees were computed based on the radius and the Gcode axis values. See Travel per Revolution in the motor group. 

### Homing Settings
Please see [G2 Homing](Homing-Operation) for details and more help on homing settings:

<pre>
{xsv:_}  Homing Search Velocity
{xlv:_}  Homing Latch Velocity
{xlb:_}  Homing Latch Backoff
{xzb:_}  Homing Zero Backoff
</pre>

By way of example, a Shapeoko2 can be set up this way:

	Setting | Description | Example
	--------|-------------|--------------
	$ST | Switch Type | 1=NC
	$XJH | X Homing Jerk | 10000000000 (10 billion)
	$XSN | X Minimum Switch Mode | 3=limit-and-homing
	$XSX | X Maximum Switch Mode | 2=limit-only
	$XTM | X Travel Maximum | 180 mm
	$XSV | X Homing Search Velocity | 3000 mm/min
	$XLV | X Homing Latch Velocity | 100 mm/min
	$XLB | X Homing Latch Backoff | 20 mm
	$XZB | X Homing Zero Backoff | 3 mm
	||
	$YJH | Y Homing Jerk | 10000000000 (10 billion)
	$YSN | Y Minimum Switch Mode | 3=limit-and-homing
	$YSX | Y Maximum Switch Mode | 2=limit-only
	$YTM | Y Travel Maximum |  180 mm
	$YSV | Y Homing Search Velocity | 3000 mm/min
	$YLV | Y Homing Latch Velocity | 100 mm/min
	$YLB | Y Homing Latch Backoff | 20 mm
	$YZB | Y Homing Zero Backoff | 3 mm
	||
	$ZJH | X Homing Jerk | 100000000 (100 million)
	$ZSN | Z Minimum Switch Mode | 0=disabled (with NC switches it's important all unused switches are disabled)
	$ZSX | Z Maximum Switch Mode | 3=limit-and-homing
	$ZTM | Z Travel Maximum | 100 mm
	$ZSV | Z Homing Search Velocity | 1000 mm/min
 	$ZLV | Z Homing Latch Velocity | 100 mm/min
	$ZLB | Z Homing Latch Backoff | 10 mm
	$ZZB | Z Homing Zero Backoff | 5 mm
	||
	$ASN | A Minimum Switch Mode | 0=disabled 
	$ASX | A Maximum Switch Mode | 0=disabled
<br>
<br>
## System Group Settings
These are general system-wide parameters and are part of the "sys" group.
<br>
###Identification Settings

### $FB - Firmware Build number
Read-only value. Example `$fb=345.06`<br>
Indicates the build of firmware and changes frequently. Please provide this number in any communication about an issue.

### $FV - Firmware Version
Read-only value. Example `$fv=0.97`<br>
Indicates the major version of the firmware; changes infrequently. Generally all settings, behaviors and other system functions will remain the same within a version - that's why this page is useful for all 0.97 versions.

### $HP - Hardware Platform
Read-only value. Returns:
* 1 = TinyG Xmega series
* 2 for Arduino Due G2 (ARM)
* 3 for TinyG v9 G2 (ARM)

### $HV - Hardware Version
Read-write value. Used to set behaviors inside the firmware. Defaults to 8 for v8. If you have a TinyG v6 or earlier you must set this value to 6.

### $ID - Unique Board Identifier
Read-only value.

###Global System Settings

### $JA - Junction Acceleration 
In conjunction with the global $jd setting sets the cornering speed. See $jd for explanation

<pre>
$ja=50000   - 50,000 mm/min^2 - a reasonable value for a modest performance machine
$ja=200000  - 200,000 mm/min^2 - a reasonable value for a higher performance machine
</pre> 

### $CT - Chordal Tolerance
Arcs are generated as sets of very short straight lines that approximate a curve. Each line is a "chord" that spans the endpoints of that segment of the arc. Chordal tolerance sets the maximum allowable deviation between the true arc and straight line that approximates it - which will be the value of the deviation in the middle of the line / arc.

Setting chordal tolerance high will make curves "rougher", but they can execute faster. Setting them smaller will make for smoother arcs that may take longer to execute. The lower-limit of $ct is set by the minimum arc segment length, which really should not be changed (See hidden parameters).
<pre>
$ct=0.01   - Normally a good value (in mm)
</pre> 

### $ST - Switch Type
Sets the type of switch used for homing and/or limits. All switches must be of the same type (mixes are not supported).
<pre>
$st=0   - Normally Open switches (NO)
$st=1   - Normally Closed switches (NC)
</pre> 

Note that probing cycles (G38.2) will work regardless of the switch setting. Probing (currently) assumes a normally open switch in the Z minimum position. During the probe cycle switches are set to NO (and ignored). They are restored to their actual $st setting when probing is complete.

### $MT - Motor Power Timeout
Sets the number of seconds motors will remain powered after the last 'event'. E.g. set to 60 to keep motors powered for 1 minute after a move completes. Only applies to motors with power management modes that actually time out the motors (modes 2 and 3). See also $ME and $MD commands, further down this page.
<pre>
$mt=5        - Keep motors energized for 5 seconds after last movement command
$mt=1000000  - Keep motors energized for 1 million seconds after last movement command (11.57 days)
</pre> 

See also [Power Management](Power-Management)
###Communications Settings

### $EJ - Enable JSON Mode on Power Up
This sets the startup mode. JSON mode can be invoked at any time by sending a line starting with an open curly '{'. JSON mode is exited any time by sending a line starting with '$', '?' or 'h'

_Note: The two startup lines on reset will always be in JSON format regardless of setting in order to allow UIs to sync with an unknown board._

<pre>
$ej=0      - Disable JSON mode on power-up and reset
$ej=1      - Enable JSON mode on power-up and reset
</pre>

### $JV - Set JSON verbosity
Sets how much information is returned in JSON mode. If you are using JSON mode with high-speed files (many short lines at high feed rates) you probably do not full verbose mode (5). 
<pre>
$jv=0      - Silent   - No response is provided for any command
$jv=1      - Footer   - Returns footer only - no command echo, gcode blocks or messages
$jv=2      - Messages - Returns footers, exception messages and gcode comment messages
$jv=3      - Configs  - Returns footer, messages, config command body
$jv=4      - Linenum  - Returns footer, messages, config command body, and gcode line numbers if present
$jv=5      - Verbose  - Returns footer, messages, config command body, and gcode blocks
</pre>

### $JS - Set JSON syntax
Sets relaxed or strict syntax for JSON messages. See [JSON Syntax](JSON-Operation#json-syntax-option---relaxed-or-strict) for details.

### $TV - Set Text mode verbosity
Sets how much information is returned in text mode. We recommend using Verbose, except for very special cases.
<pre>
$tv=0      - Silent - no response is provided
$tv=1      - Verbose - returns OK and error responses
</pre>

### $QV - Queue Report Verbosity
Queue reports return the number of available buffers in the planner queue. The planner queue has 28 buffers and therefore can have as many as 28 Gcode blocks queued for execution. An empty queue will report 28 available buffers. A full one will report 0. 

Using the planner queue depth as a way to manage flow control when sending a Gcode file is actually a much better way than managing the serial input buffer. If you keep the planner full to about 2 blocks available it will run really smoothly. You also want to make sure the queue doesn't starve, say - more than 20 blocks available.

Verbosity settings are:
<pre>
$qv=0      - Silent - queue reports are off
$qv=1      - Single - returns reports when depth changes and is above hi water mark or below low water mark
$qv=2      - Triple - returns queue reports for every block queued to the planner buffer
</pre>

You can also get a manual queue report by sending [$qr](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration-for-Firmware-Version-0.97#qr---queue-report)

_Note: In general you don't want to fill up the buffer with more than 24 commands as some Gcode blocks can occupy multiple planner buffer slots (4 are allowed)._

### $SV - Status Report Verbosity
Please see [Status Reports](https://github.com/synthetos/TinyG/wiki/Status-Reports) for a discussion of $sv and $si status report settings.
<pre>
$sv=0      - Silent   - status reports are off
$sv=1      - Filtered - returns only changed values in status reports
$sv=2      - Verbose  - returns all values in status reports
</pre>

### $SI - Status Interval 
The minimum is 100 ms. Trying to set a value below the minimum will set the minimum value. 
<pre>
$si=250    - Status interval in milliseconds
</pre>

### $IC - Ignore CR or LF on RX 
_Note: IC was removed in 0.97. Lines may be terminated with any combination of <CR>, <LF>, <CR<LF>, <LF<CR>. It doesn't matter anymore._

### $EC - Expand LF to CRLF on TX data
If this setting is OFF returned lines have a single <LF>. If on they are terminated with <CR><LF> (2 characters).
<pre>
$ec=0      - off
$ec=1      - on
</pre> 

### $EE - Enable Character Echo 
This should be disabled for JSON mode. In text mode it's optional either way.
<pre>
$ee=0      - Disable character echo
$ee=1      - Enable character echo
</pre> 

### $EX - Enable Flow Control 
<pre>
$ex=0      - Disable flow control 
$ex=1      - Enable XON/XOFF flow control protocol 
$ex=2      - Enable RTS/CTS flow control protocol 
</pre>

### $BAUD - Set USB Baud Rate
The default baud rate for the USB port is 115,200 baud. The following additional baud rates may be set. The sequence for changing the baud rate is: (1) Issue the $baud command, (2) wait for a response verifying the command, (3) change to the new baud rate.
<pre>
$baud=0     - Illegal baud rate setting. Returns an error
$baud=1     - 9600
$baud=2     - 19200
$baud=3     - 38400
$baud=4     - 57600
$baud=5     - 115200
$baud=6     - 230400
</pre>

####Gcode Default Parameters
These parameters set the values for the Gcode model on power-up or reset. They do not affect the current gcode dynamic model. <br><br>
For example, entering $gun=0 will cause the system to start up from reset or power up in inches mode, but will **not** change the system to inches mode when it is entered. A G20 or G21 received in the Gcode stream will change the units to inches or MM mode, respectively. On reset or restart they will change back to the $gun setting.

These parameters are reported as part of the "sys" group.

### $GPL - Gcode Default Plane Selection
<pre>
$gpl=0      - G17 (XY plane)
$gpl=1      - G18 (XZ plane)
$gpl=2      - G19 (YZ plane)
</pre> 

###$GUN - Gcode Default Units
<pre>
$gun=0      - G20 (inches)
$gun=1      - G21 (millimeters)
</pre> 

###$GCO - Gcode Default Coordinate System
<pre>
$gco=1      - G54 (coordinate system 1)
$gco=2      - G55 (coordinate system 2)
$gco=3      - G56 (coordinate system 3)
$gco=4      - G57 (coordinate system 4)
$gco=5      - G58 (coordinate system 5)
$gco=6      - G59 (coordinate system 6)
</pre> 

###$GPA - Gcode Default Path Control
<pre>
$gpa=0      - G61 (exact path mode)
$gpa=1      - G61.1 (exact stop mode)
$gpa=2      - G64 (continuous mode)
</pre> 

### $GDI - Gcode Distance Mode
<pre>
$gdi=0      - G90 (absolute mode)
$gdi=1      - G91 (incremental mode)
</pre> 

## Coordinate System and Origin Offsets 
### $g54x - $g59c
Coordinate system offsets are the values used by G54, G55, G56, G57, G58 and G59 commands to define the offsets from the machine (absolute) coordinate system for X,Y,Z,A,B and C. G54-G59 correspond to the Gcode coordinate systems 1-6, respectively. 

By convention G54 is often set to no offsets (all zeroes) so it is the same as the machine's absolute coordinate system. This is true because the G53 command "move in absolute coordinates" is only in effect for the current Gcode block. After that the dynamic model reverts to the coordinate system previously in effect. So if you want to say in absolute coordinates you need a persistent machine coordinate system, by convention G54.

Another convention is to set G55 to your common coordinate system, we set this to be 0,0 in the middle of the table. So once you have zeroed issuing g55 g28 will set to this system and position the head in the middle of the table. (Note: this can be done on one line of gcode - it does not need to be 2 separate commands).

G54-G59 offsets can be set per the following example:
<pre>
$g54x=0         Set G54 to be the same as the machine coordinate system
$g54y=0
$g54z=0
$g54a=0
$g54b=0
$g54c=0

$g55x=90.0      Set G55 to be in the middle of the table
$g55y=90.0
$g55z=0
$g55a=0
$g55b=0
$g55c=0
</pre> 

In JSON mode you can set a coordinate system in a single command. Only those axes specified are changed. 
<pre>
{"g55"":{"x":90,"y":90,"z":"0"}}
</pre> 

#### Displaying Offsets
Offsets can be displayed individually

`$g54x` - returns a single value 

...or as a group: 
`$g54` - returns all 6 values in the G54 group 
`$g92` - returns all 6 values of the origin offset group 

...or all together: 
`$o` - returns all offsets in the system (not available in JSON)

Note: the G54-G59 settings are persistent settings that are preserved between resets (i.e. in EEPROM), unlike the G92 origin offset settings and the G28 and G30 "go home" settings which are just in the volatile Gcode model and are thus not preserved. 

#### G10 Operation
Gcode provides the G10 L2 command to perform this same coordinate system offset setting function. Coordinate offsets can be set from Gcode using the G10 command, e.g. G10 P2 L2 X20.000 - the P word is the coordinate system numbered 1-6, the L word =2 is according to standard, but is ignored by TinyG (for now)

TinyG does not persist G10 settings, however. This is not in accordance with the [Gcode spec](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CEkQFjAA&url=http%3A%2F%2Fciteseerx.ist.psu.edu%2Fviewdoc%2Fdownload%3Fdoi%3D10.1.1.141.2441%26rep%3Drep1%26type%3Dpdf&ei=rm7UULm2F6ex0AH1sICQDA&usg=AFQjCNH72m_sRg2TycD-8cf8d40FZ6Co_g&sig2=MrjWabHA5YwPsWtrj2TtOA&bvm=bv.1355534169,d.dmQ). Any G10 settings that are provided will be used until reset, power cycle, or they are overwritten by a $g5xx command or another G10 command. 

## Commands
These commands cause various actions, and are not technically "settings".

### $SR - Status Report
Returns a status report or set the contents of a status report (JSON only). Identical to ? command. See [Status Reports]() for details.

### $QR - Queue Report
Manually request a queue report. See [$QV](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration#qv---queue-report-verbosity) for details.

### $QF - Queue Flush
Removes all Gcode blocks remaining in the planner queue. This is useful to clear the buffer after a feedhold to create homing, jogging, probes and other cycles.

### $MD - Motor De-energize
Unpower all motors. See [Power Management](Power-Management)

### $ME - Motor Energize
Energizes all non-disabled motors. Motors will time out according to their power management a mode and timeout value. See [Power Management](Power-Management)

### $TEST - Run Self Test
Execute `$test` to get a listing of available tests. Run `$test=N`, where N is the test number.

### $DEFA - Reset default profile settings
TinyG comes with a set of defaults pre-programmed to a specific machine profile. The default profile is set for a relatively slow screw machine such as the Zen Toolworks 7x12. Other default profiles are settable at compile time by including the right settings_XXXXXX.h file. If you are having trouble with your settings and want to revert to the default settings enter: `$defa=1`  

***WARNING*** This will revert all settings to defaults. Do a screencap of the $$ dump if you want to refer back to the current settings

## Hidden Parameters
These parameters are not part of any group and generally should not be changed. Serious malfunction can occur if these are not set correctly

### $ML- Minimum Line Segment 
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>
$ml=0.08    - Do not change this value
</pre> 

### $MA - Minimum Arc Segment 
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>$ma=0.10    - Do not change this value
</pre> 

### $MS - Minimum Segment time in microseconds - Refers to S-curve interpolation segments
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>
$ms=5000  - Do not change this value
</pre> 