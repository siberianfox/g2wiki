_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

**!!!! TAKE NOTE: The 3D printer extensions documented on this page are pre-release and experimental (hence the leading underscore). These settings will NOT be present in the release ultimate 3D printing releases. Please expect any UIs or code written to these specifications to require changes. The settings on this page will be removed and will not be backwards compatible. !!!!**

# Heater Groups

This page documents axis settings. Each axis object ("group") has a collection of parameters for that axis. There are 6 axis groups, X,Y,Z linear axes, and A,B,C rotary axes. Not all axes have all parameters.

- Axis examples use X axis, but any axis is OK unless otherwise noted

## Summary
There are 3 heater groups. The examples below show heater group 1 - `{he1:n}`

	Setting | Description | Notes
	--------|-------------|-------
	[{_he1e:...}](#_he1e---heater-enable) | Heater Enable | 0=off, 1=on 
	[{_he1p:...}](#_he1p---heater-p) | Heater P (PID) | read/write 
	[{_he1i:...}](#_he1i---heater-i) | Heater I (PID) | read/write 
	[{_he1d:...}](#_he1d---heater-d) | Heater D (PID) | read/write 
	[{_he1st:...}](#_he1st---set-temperature) | Set setpoint temperature | write-only
	[{_he1t:...}](#_he1t---get-temperature) | Current temperature | read-only
	[{_he1op:...}](#_he1op---heater-output) | Heater PWM level | read-only
	[{_he1tr:...}](#_he1tr---thermistor-resistance) | Thermistor resistance | read-only
	[{_he1at:...}](#_he1at---at-temperature) | "At temperature" flag | read-only 
	[{_he1an:...}](#_he1an---heater-adc) | Heater ADC reading | read-only
	[{_he1fp:...}](#_he1fp---fan-power) | Fan power | read/write
	[{_he1fm:...}](#_he1fm---fan-min-power) | Fan min power | read/write
	[{_he1fl:...}](#_he1fl---fan-low-temperature) | Fan low temperature | read/write
	[{_he1fh:...}](#_he1fh---fan-high-temperature) | Fan high temperature | read/write

### {he1:{e:...}} - Heater Enable
Sets the function of the axis.

- {xam:0}  Disable. All input to that axis will be ignored and the axis will not move. 
- {xam:1}  Standard. Linear axes move in length units. Rotary axes move in degrees. 
- {xam:2}  Inhibited. Axis values are taken into account when planning moves, but the axis will not move. Use this to perform a Z kill or to do a compute-only run.
- {xam:3}  Radius mode. (Rotary axes only) In radius mode gcode values are interpreted as linear units; either inches or mm depending on the prevailing G20/G21 setting. The conversion of linear units to degrees is accomplished using the radius setting for that axis. See $aRA for details. 

### xvm - Velocity Maximum
(aka traverse rate). Sets the maximum velocity the axis will move during a G0 traverse move. This is set in length units per minute for linear axes, degrees per minute for rotary axes. 

Note that the max velocity is *per-axis*. Diagonal / multi-axis traverses will actually occur at the fastest speed the combined set of axes and the geometry will allow, and may be faster than the individual axis max velocities. For example, max velocity for X and Y are set to 1000 mm/min. For a 45 degree traverse in X and Y the toolhead would travel at 1414.21 mm/min. 

<pre>
{xvm:1200}   set X maximum velocity (G0) to 1200 mm/min
{avm:36000}  set A to 100 revolutions per minute (360 * 100)
{zvm:30.0}   set Z to 30 inches per minute - in G20 / inches mode
</pre>
 
### xfr - Feed Rate Maximum
Sets the maximum velocity the axis will be allowed to move during a feed in a G1, G2, or G3 move. This works similarly to maximum velocity, but instead of actually setting the speed, it only serves to establish a "do not exceed" for Gcode F words. Put another way, the maximum feed rate setting is NOT used to set the Gcode's F value; it is only a maximum that may be used to limit the F value provided in a gcode file.

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

### ara - Radius value
The radius value is used by rotational axes only (A, B and C) to convert linear units to degrees when in radius mode. 

For example; if the A radius is set to 10 mm it means that a value of 62.8318531 mm will make the A axis travel one full revolution - as 62.383... is the circumference of the circle of radius R ( 2*PI*R, or 10 * 2 * 3.14159...) (Assuming nTR = 360 -- see note below). Receiving the gcode block `G0 A62.83` will turn the A axis one full revolution (360 degrees) from a starting position of 0. All internal computations and settings are still in degrees - it's just that gcode units received for the axis are converted to degrees using the specified radius. 

Note that the Travel per Revolution value (1tr) is used but unaffected in radius mode. The degrees per revolution still applies, it's just that the degrees were computed based on the radius and the Gcode axis values. See Travel per Revolution in the motor group. 

# Homing Settings
Please see [g2core Homing](Homing-g2core) for details and more help on homing settings:

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