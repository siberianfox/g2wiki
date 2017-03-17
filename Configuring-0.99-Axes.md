_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

# Axis Groups

This page documents axis settings. Each axis object ("group") has a collection of parameters for that axis. There are 6 axis groups, X,Y,Z linear axes, and A,B,C rotary axes. Not all axes have all parameters.

- Axis examples use X axis, but any axis is OK unless otherwise noted

## Summary

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

_Note: The behavior of JM has changed in 0.99. All values are multiplied by 1,000,000 by default_

Sets the maximum jerk value for that axis. Jerk is settable independently for each axis to support machines with different dynamics per axis - such as Shapeoko2 with belts for X and Y, screws for Z, Probotix with 5 pitch X and Y screws and 12 pitch Z screws, and any machine with both linear and rotary axes.

Jerk is in units per minutes cubed, so the numbers tend to be quite large - typically in the many millions or even billions of  mm/min^3. So all entries are considered to be in millions. e.g. a value of 500 is actually 500,000,000 mm/min^3. See examples below. Jerk values are also displayed in this divide-by-million form.

<pre>
{xjm:50}  Set X jerk to 50 million MM per min^3. This is a good value for a slow machine
{xjm:5000}      X jerk for Shapeoko. Yes, that's 5 billion
</pre> 

The jerk term in mm is measured in mm/min^3. In inches mode it's units are inches/min^3. So the conversion from mm to inches is 1/(25.4). The same values as above are shown in inches are: 
<pre>
50 M mm/min^3     is 1.968504 M in/min^3, so 2.0 would suffice
25 M mm/min^3     is 984,251 M in/min^3, so 1.0 would suffice
5000 M mm/min^3   is 196.8504 M in/min^3, so 200 would suffice
</pre> 

### xjh - Jerk High

_Note: The behavior of JH has changed in 0.99. All values are multiplied by 1,000,000 by default_

Sets the jerk value used for homing to stop movement when switches are hit or released. You generally want this value to be somewhat larger than the xJM value, as this determines how fast the axis will stop once it hits a limit or homing switch. You generally want this as fast as you can get it without losing steps on the accelerations.

The JM discussion above applies. 

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
$XJH | X Homing Jerk | 10000 (10 billion)
$XSN | X Minimum Switch Mode | 3=limit-and-homing
$XSX | X Maximum Switch Mode | 2=limit-only
$XTM | X Travel Maximum | 180 mm
$XSV | X Homing Search Velocity | 3000 mm/min
$XLV | X Homing Latch Velocity | 100 mm/min
$XLB | X Homing Latch Backoff | 20 mm
$XZB | X Homing Zero Backoff | 3 mm
|____|
$YJH | Y Homing Jerk | 10000 (10 billion)
$YSN | Y Minimum Switch Mode | 3=limit-and-homing
$YSX | Y Maximum Switch Mode | 2=limit-only
$YTM | Y Travel Maximum |  180 mm
$YSV | Y Homing Search Velocity | 3000 mm/min
$YLV | Y Homing Latch Velocity | 100 mm/min
$YLB | Y Homing Latch Backoff | 20 mm
$YZB | Y Homing Zero Backoff | 3 mm
|____|
$ZJH | X Homing Jerk | 100 (100 million)
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