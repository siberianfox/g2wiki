_This page documents digital inputs for 0.98. Note that in 0.98 the generalized GPIO system is only available for inputs. Generalized outputs are not supported until 0.99_

To read the firmware version type `{fv:n}`. To read build number type `{fb:n}`

Related Pages:
- [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)
- [Configuring Digital Inputs on TinyGv9 Boards](TinyGv9-Page)

##Digital Inputs
### Reading Digital Inputs
The current state of an input can be read using JSON objects. `{inN:n}` will return 0 if inactive, 1 if active (tripped), and `null` if the input is disabled or not present (large numbers may return status code 100). Note that the 0/1 return states are corrected for input sense. 1 is active. Digital inputs are read-only.
<pre>
{in1:n}  Read digital input 1, Returns 0, 1, or null
{in2:n}
...
{inN:n}
{in:n}   Read a all digital inputs as a single JSON object
</pre>

It’s possible to register inputs (and outputs) in the status report (set to filtered, please) and detect switch state changes. If you only want switch state reports but have no other effect, select action=none and function=none. The switch closure will still be available to the switch read routines.

_Note: Currently the readout `{inN:n}` is the same number as the configurator `{diN..:}`. In future releases these will be mappable._

### Configuring Digital Inputs
Digital inputs are configured using a set of digital input objects referenced as:
<pre>
{di1:n}  Group of parameters for digital input 1
{di2:n}
...
{diN:n}
{d1:n}   Uber-group of all digital input groups
</pre>

Digital inputs have these attributes (using di1 as an example)

	Name | Description | Values
	------|------------|---------
	{di1mo: | mode | 0=active low (NO), 1=active high (NC)
	{di1ac: | action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=panic, 5=reset
	{di1fn: | function | 0=none, 1=limit, 2=interlock, 3=shutdown

Inputs are sensitive to the leading edge of the transition – so falling edge for NO and rising for NC. When an input triggers it enters a lockout state for some period of time where it will not trigger again (a deglitching mechanism). Typically about 50 ms.

- STOP is a deceleration to zero at the axes normal jerk value {xjm}
- FAST_STOP is a high speed stop at the high-speed jerk value {xjh}. High speed jerk may be too high to start the motor, but can be used to stop it without losing position
- HALT stops immediately without regard to deceleration. Position may be lost.
- PANIC stops all motors and devices immediately and puts the machine in a PANIC state. (This will move to Function in later release)
- RESET resets the board if the input triggers

The function is the default function for that input. These functions set flags that are executed by the callbacks in the main loop. The function will be called unless an override for that function is in effect (e.g. limit override). Please see [Alarm Processing](Alarm-Processing) for more details. Functions include:

- LIMIT acts as a limit switch which goes into an ALARM state. Enter {clear:n} to clear
- INTERLOCK pauses all movement until the interlock input is restored
- SHUTDOWN puts the machine into a shutdown state. It must be recovered with a reset or power cycle
- ALARM (will be added in later release)
- PANIC (will be added in later release)

Internal state for inputs may include:

	Name | Description 
	------|------------
	lockout_timer | time in ticks (ms) to remove lockout
	homing_mode | set true if the switch is the homing switch now

### Digital Inputs During Homing
Homing is an exception as an input can be configured as a homing input and a limit input. To configure homing these new parameters are added to the axis config:

	Name | Description | Values
	------|------------|---------
	{xhi: | homing input | 1-N corresponding to input switch, or 0 to disable homing for this axis
	{xhd: | homing direction | 0=search-to-negative, 1=search-to-positive

Inputs operate differently during homing – sequence is:
- Limits are overridden so that all limits are inactive
- The axis being homed sets the input to “homing_mode”
- The homing function attempts to clear off the homing switch if it is depressed
- The rest is normal – search, latch, zero backoff, set zero, set axis to homed state. Uses HALT jerk value `{xjh:...}`
- At the end of homing limit override is removed

Notes and questions:
- The above works for dedicated homing switches. It will also work for shared homing switches except for the initial backoff (off a homing switch), which will somehow need to be disabled.
- Note – this does not address dual-axis homing such as squaring a dual-gantry Y

### Digital Inputs During Probing
Probing is also an exception. Currently probing can only be performed on the Zmin input (di5 on the v9). Di5 should be set as Normally Open (Active Low).

#Digital Outputs
### Reading Digital Outputs
The current state of an output can be read or written using JSON objects. `{outN:n}` will return 0 if inactive, 1 if active (tripped), and `null` if the output is disabled or not present. Note that the 0/1 values are corrected for output sense - 1 is active.

<pre>
{out1:n}	output state
{out2:n}	etc…
{out:n}		returns all outputs in a single JSON object
</pre>

It’s possible to register outputs in the status report (set to filtered, please) and detect output state changes.

_Note: Currently the readout `{outN:n}` is the same number as the configurator `{doN..:}`. In future releases these will be mappable._

### Configuring Digital Outputs
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
	{do1mo: | mode | 0=active low, 1=active high


==v9 Digital Inputs

.Digital Input Mapping to v9 Connectors
|===
| Input | Label | Input Mode | Action | Function | Notes 
| DI1 | Xmin | Normally Closed | Stop | Limit |
| DI2 | Xmax | Normally Closed | Stop | Limit |
| DI3 | Ymin | Normally Closed | Stop | Limit |
| DI4 | Ymax | Normally Closed | Stop | Limit |
| DI5 | Zmin | Active High | None | None | Also used as Z probe
| DI6 | Zmax | Disabled | Stop | Limit |
| DI7 | Amin | Disabled | Stop | Limit | Only on J15 2x13 header
| DI8 | Amax | Disabled | Stop | Limit | Only on J15 2x13 header
| DI9 | Interlock | Active High | Halt | Shutdown | Only on J15 2x13 header
|===

.Parameter Values
|===
| Parameter | Value | Value Label | Notes 
| Input Mode | -1 | Disabled |
| | 0 | Active Low | aka Normally Open
| | 1 | Active High | aka Normally Closed
| Action | 0 | None |
| | 1 | Stop | Stop without losing position, observe jerk
| | 2 | Fast Stop | Stop without losing position, fast jerk
| | 3 | Halt | Stop immediately, may lose position
| | 4 | Reset | Reset system
| Function | 0 | None |
| | 1 | Limit | Limit has been hit, unrecoverable
| | 2 | Interlock | Interlock open, recoverable
| | 3 | Shutdown | Enter shutdown
| | 4 | Panic | Cause system panic
|===