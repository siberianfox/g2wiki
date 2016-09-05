_This page describes how digital inputs work **provisionally** in firmware version 0.99, firmware builds 100+. To read the firmware version type `{fv:n}`. To read build number type `{fb:n}`_

Related Pages:
- [Configuration for Firmware Version 0.99](Configuration-for-Firmware-Version-0.98)
- [Digital IO for Firmware Version 0.98](Digital-IO-0.98)

#Overview
Digital IO on 0.99 currently supports generalized digital inputs and generalized digital outputs. 0.99 represents a step along the way towards a fully general digital IO system that supports the creation of arbitrary devices and mapping IO to the functions in those devices. Please expect IO functionality to change in future releases, as well as some of the setup parameters.

Currently the digital inputs still accept the "old style" Mode parameter - i.e. the input enable and polarity are set by the Mode parameter (mo). In the near future 0.99 will be updated to split Mode (mo) and Polarity (po) into separate configuration variables. The documentation below has it listed both ways, with appropriate warnings.

## Overview of System Model
General Purpose IO is the basis of the g2core system model. **IO Primitive** is the lowest layer, consisting of physical input and output pins. Primitives may also include some internal code that generates and consumes **signals** and **events** (i.e. changes in the state of signals). 

**Signal** A signal is any value that is communicated. It may be binary (0,1), or analog ranging 0.000 - 1.000, for example. An input switch generates a binary signal. An analog-to-digital converter generates an analog signal. A function that waits until a heater is within regulated temperature may generate a signal as a binary output.

**Event** An event is a transition of a signal, such as an input going from inactive to its active state.

**Exposed-as** is how a primitive IO can be exposed. primitive IOs can be `inM`, `outM` and `adcM`.

**Functions** are any code that does something simple, like react to a limit switch being hit, or trigger an alarm state. A function is anything that generates or consumes a signal. A limit function consumes one or more input signals. A PWM digital output consumes an analog signal and converts it to PWM on an output pin.

**Bindings** are how you connect primitives to functions; for example a limit function can be bound to one of more inputs (primitives) that are wired to limit switches. 

A **Component** is a collection of one or more functions that does something. In some cases the component is very simple and only has one function. Like a component to read an input and report it out as, say, a person-is-at-the-door signal. In other cases a component may be more complex, encapsulating multiple functions. A heater is a good example, as it has PWM outputs, temperature inputs, output signals and a variety of settings. A heater is a component consisting of PWM, temperature sensors, PID processing, timeout functions, and maybe more. Components may be part of other components - the heater component may be part of an extruder component.


# IO Primitives
The three IO primitives are **digital input**, **digital output**, and **analog input** (analog input is not implemented in 0.99).

**Analog output** is not implemented, but practically speaking we always use pulse width modulation (PWM) for variable outputs, and PWM functionality is folded into the digital outputs.

IO Primitives used to configure IO, but not to read or set it. The value of the primitive is accessible based on its **exposed-as** object. The primitive may also be bound to functions that map (bind) to the IO primitives.

The IO primitives are exposed as per these examples:
- `di`_N_ is exposed as `in`_N_
- `do`_N_ is exposed as `out`_N_

Example: `{di1:...}` is used to configure digital input 1. To read input 1 use `{in1:n}`
Example: `{do1:...}` is used to configure digital output 1.  Read as `{out1:n}` and write as `{out1:1}`


## Digital Inputs
The current state of an input can be read using JSON objects. `{inN:n}` will return 0 if inactive, 1 if active (tripped), and `null` if the input is disabled or not present (large numbers may return status code 100). Note that the 0/1 return states are corrected for input sense. 1 is active. Digital inputs are read-only.
<pre>
{in1:n}  Read digital input 1, Returns 0, 1, or null
{in2:n}
...
{inN:n}
{in:n}   Read a all digital inputs as a single JSON object
</pre>

It’s possible to include inputs (and outputs) in the status report (set to filtered, please) and report on switch state changes. If you only want switch state reports but have no other effect, select action=none and function=none. The switch closure will still be available to the switch read routines.

In addition to any assigned function(s) inputs may have an action that occurs immediately, i.e. the action is tied to the actual pin firing interrupt. This is needed for some time-sensitive operations like machine halts.

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

**Digital Input Properties**
- The input is exposed via JSON `in`_N_ 
- The digital input value reads as `0` (off) or `1` (on)
- The value is sense-corrected according to the Polarity selected in the Mode setting
- The value is conditioned - it's deglitched, debounced or otherwise conditioned
- The exposed value is read-only - `in`_N_ cannot be written and is an error
- Attempting to read a value of a disabled (or unavailable) `in` will return a `null` value. A status code may also return an error if an unavailable or non-existent input was queried.
- Digital inputs can be waited on using the `M101` command, thus be part of the job tape and synchronized with the job. An example is waiting on an input to become active before continuing the job.

_Note: Currently the readout `{inN:n}` is the same number as the configurator `{diN..:}`. In future releases these will be mappable._


***Configuration Values*** Input pins `di`_N_ can be configured in JSON:

   Name | Description | Values
   ------|------------|---------
   `{di1mo:` | mode | 0=active_low, 1=active_hi, 2=disabled
   `{di1ac:` | action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=cycle_start, 5=alarm, 6=shutdown, 7=panic, 8=reset
 
   Name | Description | Values
   ------|------------|---------
   `{di1mo:` | mode | 0=disabled, 1=enabled
   `{di1po:` | polarity | 0=normal (active high), 1=inverted (active low)
   `{di1ac:` | action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=cycle_start, 5=alarm, 6=shutdown, 7=panic, 8=reset
   `{di1in:` | exposed-as | 0=not-exposed, 1-M = bound to `in1` through `inM`

Notes:
- Mode
  - Mode will return NULL if an input is queried that is not available due to hardware or configuration
- Actions
  - `Stop` is a controlled feedhold that preserves position
  - `Fast_Stop` is a controlled feedhold at high jerk that preserves position
  - `Halt` ceases steps immediately and does not preserve position
  - `Cycle_start` starts or resumes motion after a feedhold or stop
  - `Alarm` places the system in an alarm state (and may initiate further system actions)
  - `Shutdown` initiates a system shutdown
  - `Panic` initiates a system panic
  - `Reset` performs a hard reset of the controller
- Attempting to configure a non-existent `di` will return a `null` value and an error status code

##Digital Inputs
### Reading Digital Inputs



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
