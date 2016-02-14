*** **Note: In the middle of editing - Feb 14, 2016** ***

_This page describes the function of machine limits, homing cycles and related Gcode for version 0.98 and later versions._

#Homing Commands and Operation
##Overview
The term "homing" in this context means setting the absolute machine coordinates to a known zero location, or _zeroing the machine_. The absolute machine coordinate system (aka "absolute coordinate system", "machine coordinate system", or "G53 coordinate system") is the reference coordinate system for the machine. Work coordinate systems G54, G55, G56, G57, G58, G59 can be defined on top of G53 as offsets to the machine coordinates, and G92 can be used to put offsets on the offsets. Yes. It gets confusing. The [coordinate systems](Coordinate-Systems) page may help.

###Homing Cycles
Homing is typically performed by running a "homing cycle" that locates the Z **maximum**, X minimum, and Y minimum limits - in that order. In CNC machines Z is often set to zero at the top of travel, with all moves towards the work surface (bed) being negative. X zero is located in the left hand corner with positive travel to the right, and Y zero at the front of the machine with positive travel to the rear. Once machine zero is set work zero can be set to the middle of the table of any other location using the coordinate offsets. It's common practice to leave G54 in the homed machine coordinates and G55 used for a "centered" coordinate system.

Z is homed first so that X and Y moves will clear any obstacles that might be on the work surface. Other machine configurations may be set up for different min and max, may or may not include all axes, or may set an axis to an arbitrary coordinate location (see G28.3).

In TinyG homing is performed by running a `G28.2 X0 Y0 Z0` command (The 0's are not used, but the X Y and Z words must have some arbitrary value). 

TinyG homing uses these "non standard" Gcode functions: 

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G28.2 | _axes_ | Homing Sequence | Homes all axes present in command. At least one axis letter must be present. The value (number) must be provided but is ignored.
	G28.3 | _axes_ | Set Position | Set machine origins for axes specified. In this case the values are meaningful. This command is useful for zeroing in cases where axes cannot otherwise be homed (e.g. no switches, infinite axis, etc.)


Some limitations / constraints in TinyG homing as currently implemented:
* The homing sequence is fixed and always starts with the Z axis (if requested). The sequence runs ZXYABC, but skipping all axes that are not specified in the G28.2 command.
* Homing supports a single home position. I.e. it does not support multiple-homes such as used by dual pallet machines and other complex machining centers
* **Homing operations (G28.2 and G28.3) must not be performed in the "middle" of a Gcode file. I.e. the machine must be idle (no queued commands) before performing either operation.**

Note: In high-end CNC machines there is often no user-accessible homing cycle as machine zero is set at the factory and does not need to be set by the end user. See [Peter Smid's CNC Programming Handbook, version 2](http://books.google.com/books?id=JNnQ8r5merMC&lpg=PA444&ots=PYOFKP-WtL&dq=Smid%20version3&pg=PA447#v=onepage&q=Smid%20version3&f=false) for details. 


## Switches
### Switch Inputs
A g2 firmware build has an arbitrary number of digital input pins that may be used for homing. On the v9 there are 9 digital inputs. Arduino Due based g2 builds may be configured to have many more. 

The inputs are 3.3v logic inputs and **must not have 5v applied to them or you will burn out the inputs**. Optical inputs can also be used providing the swing between 0 and 3.3 volts.

In g2 inputs are sensitive to the leading edge of the transition – so falling edge for NO and rising for NC. When an input triggers it enters a lockout state for some period of time where it will not trigger again (a deglitching mechanism). Typically about 50 ms.

Additionally, on the TinyG v9 and some other boards these inputs are electrically de-glitched with a resistor-capacitor pair and also transient protected for electrostatic discharge. The RC circuit performs a pull-up of the signal and prevents spurious noise from getting into the line. If you are using a Due-based g2 or some other config we recommend using this circuit or something like it:

![](images/digital_input.jpg)

### Switch Wiring
To connect a switch to an input pin simply wire the switch across the ground and the input. This applies to both normally open (NO) and normally closed (NC) switches. Each switch may be selected for NO or NC independently. We recommend using NC switches for better noise immunity to prevent false firings. 

Wire a single switch to each axis that will be homed. Homing does not require each switch input to be independent - i.e. you can use a single input for multiple axes, but homing works better if the switches are independent. The following configuration is typical for most milling machines and 3D printers:

	Pin  | Function    | Position on machine
	-----|-------------|-------------------------
	Xmin | X homing switch | on the left of the machine
	Ymin | Y homing switch | at the front of the machine
	Zmax | Z homing switch | at the top of the Z axis travel

####Limit Switches
Having wired the homing inputs, any other inputs may be wired as axis limit switches (alarm) or left unused. Limit switches also share an input, but provide better alarm messages if they are independent. Adding limit switches would add these three switches to the example above:

	Pin  | Function    | Position on machine
	-----|-------------|-------------------------
	Xmax | X limit switch | on the right of the machine
	Ymax | Y limit switch | at the back of the machine
	Zmin | Z limit switch | at the bottom of the Z axis travel

## Configuring Homing
It is mandatory that the homing configuration settings match the physical switch configuration otherwise homing simply won't work. In the case of NC switches the entire machine may be rendered inoperative if these settings are not in alignment. Homing is set up by first configuring the digital inputs, then configuring each homing axis to use the proper input. 

### Configuring Digital Inputs
Digital inputs are explained on the [GPIO page](Digital-IO-(GPIO)) but are recapped here (using di1 as an example):

	Name | Description | Values
	------|------------|---------
	{di1mo:_} | input mode |-1=disabled, 0=active low (NO), 1=active high (NC)
	{di1ac:_} | input action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=reset
	{di1fn:_} | input function | 0=none, 1=limit, 2=interlock, 3=shutdown, 4=panic

For homing we recommend using "active high" (1) switches. Use "stop" (1) as the action, and "none" (0) as the function. 

### Configuring Axes for Homing 
Setting up an axis for homing is done as so (using the X axis as an example):

	Setting | Description | Notes
	--------|-------------|--------------
	{xtn:_} | Travel Minimum | Minimum travel in absolute coordinates 
	{xtm:_} | Travel Maximum | Maximum travel in absolute coordinates 
	{xjh:_} | Jerk High | Jerk used during homing operations
	{xhi:_} | Homing Input | Switch (input) to use for homing this axis
	{xhd:_} | Homing Direction | 0=search-towards-negative, 1=search-torwards-positive
	{xsv:_} | Search velocity | Homing speed during search phase (drive to switch)
	{xlv:_} | Latch velocity | Homing speed during latch phase (drive off switch)
	{xlb:_} | Latch backoff | Maximum distance to back off switch during latch phase (drive off switch)
	{xzb:_} | Zero backoff | Offset from switch for zero in absolute coordinates

* xtn/xtm - Travel minimum and maximum. The distrance between the min and max travel (plus a small factor) is used to determine how far to search for the homing switch before giving up. This is to prevent the system from grinding away indefinitely if a homing swithc is not found or is mis-configured. 
* xjh - Jerk High - Sets the jerk value used for homing to stop movement when switches are hit or released. You generally want this value to be larger than the xJM value, as this determines how fast the axis will stop once it hits the switch. You generally want this as fast as you can get it without losing steps on the accelerations.

	**$xJH** | Homing Jerk | Set this to stop quickly on switches. May need to be larger than the $xJM
	**$xSV** | Homing Search Velocity | Velocity for initially finding the homing switch. Set a moderate pace, e.g. 1/4 to 1/2 your max velocity
	**$xLV** | Homing Latch Velocity | Velocity for latching phase. Set this very low for best positional accuracy, e.g. 10 mm/min
	**$xLB** | Homing Latch Backoff | Distance to back off switch during latch and for clears. Must be large enough to totally clear the switch or you will see frustrating errors.
	**$xZB** | Homing Zero Backoff | Distance to back off switch before setting machine coordinate system zero. Usually no more than a few millimeters 

**Note:** Min and max travel are used for two functions (1) setting [soft limit](Homing-and-Limits-Setup-and-Troubleshooting#soft-limits) boundaries, and (2) they are added together to determine the total travel that an axis can move in a homing operation. Typically min is set to zero and max is something (e.g. 280mm). For soft limits it can be useful to set set Z max = 0 and Zmin = -something. If these values are misconfigured the search  could stop before it reaches the intended switch, and the homing operation is canceled.

Here an example of the JSON for setting up a Shapeoko2, dual Y axis

//*** Input / output settings ***
/*
    IO_MODE_DISABLED
    IO_ACTIVE_LOW    aka NORMALLY_OPEN
    IO_ACTIVE_HIGH   aka NORMALLY_CLOSED

    INPUT_ACTION_NONE
    INPUT_ACTION_STOP
    INPUT_ACTION_FAST_STOP
    INPUT_ACTION_HALT
    INPUT_ACTION_RESET

    INPUT_FUNCTION_NONE
    INPUT_FUNCTION_LIMIT
    INPUT_FUNCTION_INTERLOCK
    INPUT_FUNCTION_SHUTDOWN
    INPUT_FUNCTION_PANIC
*/
// Xmin on v9 board
#define DI1_MODE                    NORMALLY_CLOSED
//#define DI1_ACTION                  INPUT_ACTION_STOP
#define DI1_ACTION                  INPUT_ACTION_NONE
#define DI1_FUNCTION                INPUT_FUNCTION_LIMIT

// Xmax
#define DI2_MODE                    NORMALLY_CLOSED
//#define DI2_ACTION                  INPUT_ACTION_STOP
#define DI2_ACTION                  INPUT_ACTION_NONE
#define DI2_FUNCTION                INPUT_FUNCTION_LIMIT

// Ymin
#define DI3_MODE                    NORMALLY_CLOSED
//#define DI3_ACTION                  INPUT_ACTION_STOP
#define DI3_ACTION                  INPUT_ACTION_NONE
#define DI3_FUNCTION                INPUT_FUNCTION_LIMIT

// Ymax
#define DI4_MODE                    NORMALLY_CLOSED
//#define DI4_ACTION                  INPUT_ACTION_STOP
#define DI4_ACTION                  INPUT_ACTION_NONE
#define DI4_FUNCTION                INPUT_FUNCTION_LIMIT

// Zmin
#define DI5_MODE                    IO_ACTIVE_HIGH   // Z probe
#define DI5_ACTION                  INPUT_ACTION_NONE
#define DI5_FUNCTION                INPUT_FUNCTION_NONE

// Zmax
#define DI6_MODE                    NORMALLY_CLOSED
//#define DI6_ACTION                  INPUT_ACTION_STOP
#define DI6_ACTION                  INPUT_ACTION_NONE
#define DI6_FUNCTION                INPUT_FUNCTION_LIMIT

// Amin
#define DI7_MODE                    IO_MODE_DISABLED
#define DI7_ACTION                  INPUT_ACTION_NONE
#define DI7_FUNCTION                INPUT_FUNCTION_NONE

// Amax
#define DI8_MODE                    IO_MODE_DISABLED
#define DI8_ACTION                  INPUT_ACTION_NONE
#define DI8_FUNCTION                INPUT_FUNCTION_NONE

// Hardware interlock input
#define DI9_MODE                    IO_MODE_DISABLED
#define DI9_ACTION                  INPUT_ACTION_NONE
#define DI9_FUNCTION                INPUT_FUNCTION_NONE

<pre>
{xsv:_}  Homing Search Velocity
{xlv:_}  Homing Latch Velocity
{xlb:_}  Homing Latch Backoff
{xzb:_}  Homing Zero Backoff
</pre>


**IMPORTANT**
* It is important to configure all switch pins (all 8) even if you are not using them. Configure all unused switches as Disabled. Otherwise NC configurations will not work.

* An axis should only be configured for one homing switch - it should not have two. The homing switch position (min or max) may be configured as homing-only or homing-and-limit. If there is a second switch on the opposite end it should be either disabled or configured as limit-only. If the switches are misconfigured homing will not run and you should see a [status code](TinyG-Status-Codes) indicating that switches re mis-configured.

## Homing Configuration Settings



### Homing Operation
## G28.2 - Homing Sequence (Homing Cycle)
G28.2 is used to home to physical home switches. G28.2 will find the home switch for an axis then set machine zero (absolute coordinate system zero) for that axis at an offset from the switch location. Format is: 
<pre>G28.2 X0 Y0 Z0 A0 B0 C0</pre>
Axes not present are ignored and their zero values are not changed.

For example. G28.2 X0 Y0 will home the X and Y axes only, in that order. The values provided for X and Y don't matter, but something must be present.

* G28.2 homes all axes present in the command 
 * The homing sequence progresses through each axis provided in the G28.2 block in turn - i.e. it does not home on multiple axes simultaneously. 
 * Axes are always executed in order of ZXYABC. The order the axis words occur in the G28.2 block has no effect. 
* Homing begins by checking the pre-conditions for homing
 * One and only one switch must be configured as the axis' homing switch
 * $xSV search velocity must be non-zero
 * $xLV latch velocity must be non-zero
 * $xLB latch backoff must be a positive number or zero
 * $xTN and $xTM - travel min and max must add to a non-zero total travel
 * Failure of any of these conditions will cancel the homing operation and return status code 240 and a message indicating the offending axis
* The following is performed for each specified axis:
 * Homing begins by testing for pre-existing switch closure for the current axis. If a switch is tripped the axis will back off the switch by the Latch Backoff ($xLB) distance.
 * Once the switches are cleared a search move is executed. The search will travel at the Search Velocity ($xSV) for total travel distance ($xTN + $xTM) towards the homing switch. The search runs until the homing switch is hit or the maximum travel is performed. 
    * If maximum travel is reached without hitting the homing switch the homing operation is canceled (check your $xTN and $xTM settings)
 * Once the switch is hit a Latch Backoff move is performed. This backs off the switch until the switch opens again. This is the repeatable zero location. 
   * If the switch does not open the latch backoff distance may be reached. This should not happen - check your latch backoff setting if the switch does not open.
 * Once the latch backoff is complete the axis moves further off the switch by the Zero Backoff amount and sets zero for that axis. The axis is marked as HOMED in the homing state variable ($homx, where 'x' is any axis).
* Once all axes are processed the affected axes will be at the absolute home location (machine zero). 
  * At this point the global homing state ($home) will indicate that the machine has been homed. Homing state can be read for global, and for each axis ($homx, $homy...), or all together using the homing group: $hom. This returns 0 or 1 for each axis to indicate homing state for each axis and the global homing state.

See also: 
* [The top of this page](Homing-and-Limits-Description-and-Operation)
* [Homing Setup and Troubleshooting page](Homing-and-Limits-Setup-and-Troubleshooting)


## G28.3 - Set Absolute Position 
G28.3 allows you to set a zero (or other value) for any axes. Some axes cannot be homed. They either don't have switches, are infinite axes like rollers on the Othercutter, or some other reason. Do the following to set a zero - for example:
<pre>
g28.3 y0
</pre> 

G28.3 also supports setting to non-zero values, if that's useful. G28.3 affects the $hom group - any axis set by g28.3 is considered set for $hom

### TinyG v7 Switch Port
TinyG v7 has 8 switch pin pairs and a 3.3v pair take-off located on the J13 jumper next to the reset button.  The switch pairs are labeled:

	Pair  | Notes
	-----|-------------
	3.3v | This pair is located on the J13 connector closest to the corner of the board
	Xmin | Corresponding switch is typically positioned at left-most travel of the machine
	Xmax | Switch typically at right-most travel
	Ymin | Switch typically at front of machine 
	Ymax | Switch typically at rear of machine 
	Zmin | Switch typically at minimum height of Z travel or omitted
	Zmax | Switch typically at maximum height of Z travel
	Amin | Most of the time A is infinite and not homed. This position can be used for a machine kill
	Amax | Ditto

For each switch pair the pin closest to the board edge is the ground, the pin next to it is the switch input as labeled on the silkscreen. The inputs are 3.3v logic inputs and **must not have 5v applied to them or you will burn out the inputs**. The inputs are tied high - with strong pullups for v7 boards and on-chip weak pullups for earlier boards. 



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