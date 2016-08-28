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
	[{ara:_}](#ara---radius-value) | Radius setting | Artificial radius to convert linear values to degrees. ABC axes only.
	[{xhi:_}](#homing-settings) | Homing Input | Switch (input) to use for homing this axis
	[{xhd:_}](#homing-settings) | Homing Direction | 0=search-towards-negative, 1=search-torwards-positive
	[{xsv:_}](#homing-settings) | Search velocity | Homing speed during search phase (drive to switch)
	[{xlv:_}](#homing-settings) | Latch velocity | Homing speed during latch phase (drive off switch)
	[{xlb:_}](#homing-settings) | Latch backoff | Maximum distance to back off switch during latch phase (drive off switch)
	[{xzb:_}](#homing-settings) | Zero backoff | Offset from switch for zero in absolute coordinates
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
Please see [G2 Homing](Homing-g2) for details and more help on homing settings:

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