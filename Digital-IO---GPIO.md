Configuring Inputs
Digital inputs are controlled using a set of digital input objects referenced as so:
di1
di2
…
diN

The group of all digital inputs can be referenced as: `di`

Digital inputs have these attributes (using di1 as an example)
Name	Description	Values
{di1mo:	mode	-1=disabled, 0=active low (NO), 1=active high (NC)
{di1ac:	action	0=none, 1=stop, 2=fast_stop, 3=halt, 4=reset
{di1fn:	function (default)	0=none, 1=limit, 2=interlock, 3=shutdown, 4=spindle_ready, 5=temperature_ready, 6=M66

Inputs are sensitive to the leading edge of the transition – so falling edge for NO and rising for NC. When an input triggers it enters a lockout state for some period of time where it will not trigger again (a deglitching mechanism). Typically 100 – 250 ms.

A STOP is a deceleration to zero at the axes normal jerk value {xjm}. A FAST_STOP is a high speed stop at the high-speed jerk value {xjh}. High speed jerk may be too high to start the motor, but can be used to stop it without losing position. HALT stops immediately without regard to deceleration. Expect position to be lost. RESET resets the board if the input triggers.

The function is the default function for that input. These functions set flags that are executed by the callbacks in the main loop. The function will be called unless an override for that function is in effect (e.g. limit override).

Internal state for inputs may include:
lockout_timer		time in ticks (ms) to remove lockout
homing_mode		set true if the switch is the homing switch now

Homing
Homing is an exception as an input can be configured as a homing input and a limit input. To configure homing these new parameters are added to the axis config:
Name	Description	Values
{xhd:	homing direction	0=search_negative, 1=search_positive
{xhi:	homing input	1-N corresponding to input, or 0 to disable homing for this axis

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

Configuring Outputs
Digital outputs are controlled using a set of digital output objects referenced as so:
do1
do2
…
doN

The group of all digital inputs can be referenced as: `do`

Digital outputs have these attributes (using do1 as an example)
Name	Description	Values
{do1st:	sense type	0=active low, 1=active high
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
