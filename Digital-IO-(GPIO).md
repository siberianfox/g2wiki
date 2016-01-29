This page describes how digital inputs work in firmware version 0.98. To report the version use {fv:n}. G2 from build 87.01 on is version 0.98. Other pages that may be of interest:

* [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)
* [Configuring Digital Inputs on TinyGv9 Boards](TinyGv9-Page)

##Configuring Inputs
Digital inputs are controlled using a set of digital input objects referenced as:
<pre>
{di1:n}
{di2:n}
...
{diN:n}
{d1:n}   Group of all digital inputs
</pre>

The state of IO can be read by separate variables. 0 = inactive, 1 = active (tripped). Note that these are corrected for input sense. Disabled inputs are returned as -1.
<pre>
{in1:n}
{in2:n}
...
{inN:n}
{in:n}   Group of all digital inputs as a single JSON object
</pre>

Digital inputs have these attributes (using di1 as an example)

	Name | Description | Values
	------|------------|---------
	{di1mo: | mode |-1=disabled, 0=active low (NO), 1=active high (NC)
	{di1ac: | action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=reset
	{di1fn: | function | 0=none, 1=limit, 2=interlock, 3=shutdown, 4=panic

Inputs are sensitive to the leading edge of the transition – so falling edge for NO and rising for NC. When an input triggers it enters a lockout state for some period of time where it will not trigger again (a deglitching mechanism). Typically about 50 ms.

- STOP is a deceleration to zero at the axes normal jerk value {xjm}
- FAST_STOP is a high speed stop at the high-speed jerk value {xjh}. High speed jerk may be too high to start the motor, but can be used to stop it without losing position
- HALT stops immediately without regard to deceleration. Position may be lost.
- RESET resets the board if the input triggers

The function is the default function for that input. These functions set flags that are executed by the callbacks in the main loop. The function will be called unless an override for that function is in effect (e.g. limit override). Please see [Alarm Processing](Alarm-Processing) for more details. Functions include:

- LIMIT acts as a limit switch which goes into an ALARM state. Enter {clear:n} to clear
- INTERLOCK pauses all movement until the interlock input is restored
- SHUTDOWN puts the machine into a shutdown state. It must be recovered with a reset or power cycle
- PANIC puts the machine into a panic state. It must be recovered with a reset or power cycle

Internal state for inputs may include:

	Name | Description 
	------|------------
	lockout_timer | time in ticks (ms) to remove lockout
	homing_mode | set true if the switch is the homing switch now

##Homing
Homing is an exception as an input can be configured as a homing input and a limit input. To configure homing these new parameters are added to the axis config:

	Name | Description | Values
	------|------------|---------
	{xhd: | homing direction | 0=search_negative, 1=search_positive
	{xhi: | homing input | 1-N corresponding to input, or 0 to disable homing for this axis

Inputs operate differently during homing – sequence is:
- Limits are overridden so that all limits are inactive
- The axis being homed sets the input to “homing_mode”
- The homing function attempts to clear off the homing switch if it is depressed
- The rest is normal – search, latch, zero backoff, set zero, set axis to homed state. Uses HALT jerk value
- Homing_mode is removed from the switch
- At the end of homing limit override is removed.

Notes and questions:
-	The above works for dedicated homing switches. It will also work for shared homing switches except for the initial backoff (off a homing switch), which will somehow need to be disabled.
-	Note – this does not address dual-axis homing such as squaring a dual-gantry Y. Do we need to discuss this?

##Probing
Probing is also an exception. Currently probing can only be performed on the Zmin input (di5 on the v9). Di5 should be set as Normally Open (Active Low).

#Configuring Outputs
(Not yet implemented)
Digital outputs are controlled using a set of digital output objects referenced as:
<pre>
{do1:n}
{do2:n}
…
{doN:n}
{do:n}     Group of all digital outputs
</pre>

Digital outputs have these attributes (using do1 as an example)

	Name | Description | Values
	------|------------|---------
	{do1st: | sense type | 0=active low, 1=active high
<tbd>

Reading IO
The state of IO can be read by separate variables. 0 = inactive, 1 = active (tripped). Note that these are corrected for input sense

{in1:n}		input state
{in2:n}		etc…
{in:n}		returns all inputs in a single JSON object

{out1:n}	output state
{out2:n}	etc…
{out:n}		returns all outputs in a single JSON object

It’s possible to register inputs (and outputs) in the status report (set to filtered, please) and detect switch state changes. If you only want switch state reports but have no other effect, select action=none and function=none. The switch closure will still be available to the switch read routines.


# Discussion of future changes (pending review)

We have a few things we need to resolve:
- Instead of having a direct mapping between `di0` and `in0`, we would like to assign an arbitrary `di` to an arbitrary `in`.
  - For example, we could have several machines where `in0` is always valid and serves the same function, but on one machine it's attached to `di0` but on another it's attached to `di5`.
- We would like to be able to assign an arbitrary output to an arbitrary function.
  - For example, we would like to say that "Spindle Speed" for "Tool 0" is actually using `do0`. Or we could assign it to `di1`.
  - The pins need to have the capabilities necessary for those functions. Beyond the obvious of an input not being an output, we would also have some functions need PWM output capability.
- Some functionality will simply NOT be reconfigurable. We cannot reassign motor pins, for example. All functions that are related to a "tool" should be reassignable, as well as some functions that are general, such as coolant.

High-level function, and the input and output they would need:
- **Spindle**
  - Controlled by `M3`, `M4`, `M5`, and `M6`, along with feed hold and other functions.
  - _Tool specific:_ Y
  - _Functions:_
    - Speed (Output w/PWM)
    - On/Off (Output)
    - Direction (Output)

- **Generic Output**
  - Controlled by JSON as `out`_N_, directly or with `M100`
  - _Tool specific:_ Optional
  - _Functions:_
    - Output

- **Generic Input**
  - Read by JSON either directly as `in`_N_ or placed in the SR filter list.
  - `M101` waits can wait for these.
  - _Tool specific:_ Optional
  - _Functions:_
    - Input
