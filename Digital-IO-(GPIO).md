This page describes how digital inputs work in firmware version 0.98, firmware build 100+. To report the version use `{fv:n}`. To report build number use `{fb:n}`

Related Pages:
- [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)
- [Configuring Digital Inputs on TinyGv9 Boards](TinyGv9-Page)

##Digital Inputs
### Reading Digital Inputs
The current state of an input can be read using JSON objects. `{inN:n}` will return 0 if inactive, 1 if active (tripped). Note that these are corrected for input sense.
<pre>
{in1:n}  Read digital input 1
{in2:n}
...
{inN:n}
{in:n}   Read a all digital inputs as a single JSON object
</pre>

### Configuring Digital Inputs
Digital inputs are configured using a set of digital input objects referenced as:
<pre>
{di1:n}  Group of parameters for digital input 1
{di2:n}
...
{diN:n}
{d1:n}   Uber-group of all digital input groups
</pre>

_Note: Currently the readout `{inN:n}` is the same number as the configurator `{diN__:}`. In future releases these will be mappable._

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
<tbd>

###Reading IO
The state of IO can be read by separate variables. 0 = inactive, 1 = active (tripped). Note that these are corrected for input sense
<pre>
{in1:n}		input state
{in2:n}		etc…
{in:n}		returns all inputs in a single JSON object

{out1:n}	output state
{out2:n}	etc…
{out:n}		returns all outputs in a single JSON object
</pre>

It’s possible to register inputs (and outputs) in the status report (set to filtered, please) and detect switch state changes. If you only want switch state reports but have no other effect, select action=none and function=none. The switch closure will still be available to the switch read routines.


# Discussion of future changes (pending review)

## IO System Model
### Layers
Listed from lowest layer to highest:

- **"IO Primitives"** refers to the physical layer and consists of digital inputs (`di`_N_), analog inputs (`ai`_N_), and digital outputs (`do`_N_). There can also be various "non-pin" or virtual "signals" at the io level, such as a timer expiring or a communications operation to or from a remote device (e.g. a command to turn on a temperature controller sent over some serial bus).

- **"Function"** refers to the logical layer. Functions are things like Spindle On, Limit Hit, Feedhold, etc. Primitive functions such as `in`N, `out`N and `adc`N are also defined.

- **"Tool"** is in keeping with the Gcode definition of a tool - which is something that is moved around in XYZ at the "controlled point" (refer to the NIST spec for this definition). For our purposes a tool may be any of: an end mill in a spindle, an extruder, a laser, a heated wire, etc. Gcode supports numbering tools (T words), assigning attributes to tools (Tool tables), and changing tools (M6). Some functions operate on tools. For example set-temperature-and-wait-until-at-temperature can be applied to an extruder; set-spindle-speed can be applied to a spindle and its associated cutter.

### Access and Visibility
The following is necessary to explain the various ways of sending IO commands under various circumstances. There are three "flavors" of accessing the system:

1. **"Configuration Access"** sets up the system so it can process a job. Configuration always occurs outside of a job; i.e. outside of a Gcode run. Configuration is uses raw JSON commands (or text mode $ commands), and never uses Gcode commands (G10's being a possible exception). Configuration commands flow over the "control" channel. In the case of dual-port connection the control channel is actually a separate USB port. In other cases it's a logical channel as both control and data channels share the same physical interface.

1. **"Runtime or Job Access"** are the actual Gcode to be run, aka the "tape". The only things that should be on the tape (in the Gcode) are Gcode commands that are to be sequenced during the job. This is considered the "data" channel. The IO system uses active comments to wrap JSON in the Gcode (M100 and M101), so IO commands can be inserted into the tape. Raw JSON is never used in the data channel, and is rejected if found. Isolating the configuration operations from the tape assists in making Gcode files more portable across machines and has other benefits as well.

1. **"Runtime Commands"** are commands that are delivered during a job, but are not part of the tape. Examples are feedhold, limit switch hits, and other user or machine initiated events or controls that may affect the running job. These are delivered over the control channel and are raw JSON.

### Other Discussion

- One design goal is to provide a logical-to-physical mapping of IO ports to make control code (e.g. UIs) more portable and consistent across different board and machine configurations. Instead of having a direct mapping between `di0` and `in0`, we would like to assign an arbitrary `di` to an arbitrary `in`.
  - For example, we could have several machines where `in0` is always valid and serves the same function (like a shutdown signal), but on one machine it's attached to `di0` but on another it's attached to `di5`.

- Another design goal is to be able to assign an arbitrary IO to an arbitrary function
  - For example, we would like to say that "heater PWM" for Tool 1 (extruder 1) is actually using `do0` whereas for Tool 2 it uses `do7`, and hide these details form the UI.
  - The pins need to have the capabilities necessary to support those functions. Beyond the obvious of an input not being an output, we have some functions that need PWM output capability, such as heater PWM

- Note that some functionality will simply NOT be reconfigurable. We cannot reassign motor pins, for example. All functions that are related to a "tool" should be reassignable, as well as some functions that are general, such as coolant

## IO Primitives
The three IO primitives are **digital input**, **analog input**, and **digital output**. Analog output is not implemented, but practically speaking we always use pulse width modulation (PWM) for variable outputs, so PWM functionality is folded into the digital outputs. The following is true of all types of IO:

- An IO pin can be mapped to one of the generics to read it's value - and the case of outputs - write the value. The numbers _N_ and _M_ can be different, supporting physical-to-logical mapping
  - `di`_N_ is exposed as `in`_M_ and read as `{in1:n}` (for example)
  - `ai`_N_ is exposed as `adc`_M_ and read as `{adc1:n}` (for example)
  - `do`_N_ is exposed as `out`_M_, read as `{out1:n}` and written as `{out1:t}` or `{out1:1}` (for example)

_(QUESTION: Is there a bound on M?)_

- In lieu of being exposed as one of the generics a pin may be associated with a specific function or a tool and exposed and be accessed as a member of that function. For example:
  - An input assigned to a function like safety interlock input `safe` can be read as `{safe:n}`.
  - An output assigned to a function like mist coolant `com` can be written as `{com:t}`. See M100 for use from Gcode.
  - If an io is assigned to some function in tool 3 then it can be read as `{t3:{some-function-in-tool:n}}`. If the machine is currently loaded with tool 3 (via `M6 T3`), `{some-function-in-tool:n}` acts like `{t3:{some-function-in-tool:n}}`. Similar for outputs.
  - Not all functions expose the "raw" value of an input or output. For example, when an output is assigned to the "spindle speed" function, there is no way to read or write the raw PWM duty-cycle output value. It can only be set via gcode `S` words and read via the JSON `{sps:N}` command (or text-mode equivalent).

- An input or output pin may be non-existent or may be explicitly disabled. Pins that don't exist will ignore attempts to enable them and return an ERROR status code. Disabled or non-existent pins, if read, will always read `null`. Attempts are made to prevent configuration from exposing disabled or non-existent pins, but should the pin be readable it will return `null` and an ERROR status code, and may also generate an exception report in some cases.

Now the details for each type and their properties:

### Digital Input "Pin"
- "Pin" may be a physical pin or the output from an internal signal such as a timer timeout or a function that outputs true or false
- A pin value is not directly accessible, but may be assigned to a function that is exposed via JSON, such as `in`_N_
  - Reads as boolean `True` or `False`
  - The value is read-only (`in`_N_ cannot be written and is an error)
  - The value is conditioned - i.e. it's deglitched, debounced or whatever conditioning the board provides
- Inputs may have actions that occur immediately as well its assigned function, i.e. actions that are bound to the actual pin firing interrupt. This is needed for some time-sensitive operations like machine halts.
- Digital inputs can be waited on using the `M101` command, thus be part of the job tape and synchronized with the job. An example is waiting on a heater to achieve its set temperature.
- ***Configuration Values*** Input pins can be configured in JSON via `di`_N_ as below:
 
   Name | Description | Values
   ------|------------|---------
   {di1mo | mode | -1=disabled, 0=active low (Normally Open), 1=active high (Normally Closed)
   {di1ac | action | 0=none, 1=stop, 2=fast_stop, 3=halt, 4=reset
   {di1fn | function | 0=none/other, 1=in1, 2=in2... (see Function Access)

### Analog Input "Pin"
- "Pin" may be a physical pin or an internal signal such as a function the returns a floating point value between 0.0 and 1.0
- A pin value is not directly accessible, but may be assigned to a function that is exposed via JSON, such as `adc`_N_
  - Reads as a float value from `0.0` to `1.0`
  - The value is read-only (`adc`_N_ cannot be written and is an error)
- Analog inputs cannot _directly_ be waited on by `M101`, but tools may have "wait" functions that can be used for synchronization to analog inputs. Tool functions may be associated with an analog pin, and then indirectly provide boolean values than can be waited on. For example, a Heater may use an analog input for the temperature sensor, and then a wait can be used for when the heater is at temperature.
- ***Configuration Values*** Input pins can be configured in JSON via `ai`_N_ as below:

   Name | Description | Values
   ------|------------|---------
   {ai1mo | mode | -1=disabled, 0=normal (LOW is 0.0), 1=inverted (HIGH is 0.0)
   {ai1fn | function | 0=none/other, 1=adc1, 2=adc2... (see Function Access)

### Digital Output "Pin"
- "Pin" may be a physical pin or the output may be virtual and used as an internal signal
- A pin value is not directly accessible but may be assigned to a function that is exposed via JSON, such as `out`_N_.
  - Set as boolean `True` for "Active" or `False` for "Inactive", or as a float of `0.0` through `1.0` if it has PWM capability that has been configured - with the value being the "Active" time in the cycle, depending on polarity.
  - If the pin or internal signal is binary (not PWM capable or such), then floating point set values will interpreted as the result of the boolean expression `(bool)(value >= 0.5)`.
  - Reads back the value it was set to as a float, which may not be the value given in the set.
    - For example, if the pin was set to `0.75` but is not PWM capable, it will read back as `1`.
    - If set with `true` it will return `1` and if set to `false` it will return `0`.
- Set and read values will always align, even if the sense of the pin makes the physical output inverted. IOW, if the pin is set to have the sense of "active low", and setting the pin to "true" causes a "LOW" voltage on a physical pin, it will still read back as `1` to indicate "active", reflecting the value it was set as.
- Outputs may be associated with tools. See discussion on inputs above.
- ***Configuration Values*** Output pins can be configured in JSON via `do`_N_ as below:

   Name | Description | Values
   ------|------------|---------
   {do1mo | mode | -1=disabled, 0=normal (active HIGH), 1=inverted (active LOW)
   {do1frq | function | -1=not PWM capable, 0=PWM off, >0 = PWM frequency in Hz
   {do1tn | tool number | 0=none, 1=tool1, etc.
   {do1fn | function | 0=none/other, 1=out1, 2-out2... (see Function Access)

## Function Access
### Assigning IO to Functions (Binding)
The IO primitive inputs and outputs can be assigned to functions. The __fn configuration will assign diN, aiN, doN by numbers to the generics inM, adcM and outM, respectively. 

Named functions (non-generics) assign their functions "downward" to a pin by setting a configuration value in that function. The __fn value will be set to 0 in this case. A pin cannot be assigned to a generic and a function at the same time.

### Accessing Functions via JSON
Direct access via JSON is straightforward. Examples:
- `{in1:n}` reads digital input 1
- `{out3:n}` reads digital output 3
- `{out3:t}` sets digital output 3 active
- `{out3:1}` sets digital output 3 active
- `{out3:0}` sets digital output 3 inactive
- `{com:0}` disables mist coolant
- `{htr...}` need a heater example here...

### Accessing Functions in Status Reports
Any function value can also be included in a status report. Any time the value changes it will be reported in the status report. We highly recommend using filtered status reports if this is the case. 

### Accessing Functions via Gode
Some IO commands and functions make sense to operate from within Gcode and thus be part of the job and synchronized with motion. We use JSON active comments, which is a Gcode comment that gets executed. In standard Gcode the string "MSG" occurring right after the open paren is an active comment that sends the comment to the console, e.g. `G0 X100 (MSG Hello World). JSON active comments rely on an open curly immediately after the open paren. 

- M100 ({...}) - JSON command synchronized with motion. The JSON command, presumably an output, will be executed when the planner gets to it.
- M101 ({...}) - Wait on event. The planner will hold until the JSON command, presumably an input, returns `true`.

### Function Cheat Sheet
For now we need a way to collect all the functions we want to bind to the IO. As this page evolves the major sub-systems will be broken out into their own pages describing their functions.

   Function | Notes
   ------------|---------
   no function | pin is not bound to a function
   generic digital input | {`in`_N_
   generic analog input | {`adc`_N_
   generic digital output | {`out`_N_
   feedhold input |
   limit switch input |
   homing switch input | (This one may not be needed)
   shutdown input | used to support external emergency stop
   interlock input | 
   spindle direction | 
   spindle on/off | 
   spindle speed | 
   spindle at speed |
   mist coolant on/off |
   flood coolant on/off |

### Generic IO Functions

#### Generic Digital Input
- Read by JSON as {inN:n}
- Can also be placed in status reports to monitor state changes (hopefully filtered!)
- `M101` waits can wait for these
- _Tool specific:_ Optional

    Function |  Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Input (no tool) | `M101 ({in2:t})` | Input | Input as `in2`: `{di1fn:2}`, No tool: `{di1tn:0}` | _None_
    Input (part of tool) | `M6 T4` -> `M101 ({in3:t})` | Input |  Input as `in3`: `{di1fn:3}`, Tool 4: `{di1tn:4}` | _None_

#### Generic Analog Input
- Read by JSON either directly as `ain`_N_. Value is floating point from 0.0 to 1.0.
- _Tool specific:_ No

    Function |  Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Analog Input | `{ain2:n}` | Analog Input | Input as `ain2`: `{ai1fn:2}` | _None_

## Generic Digital Output
- Controlled by JSON as `out`_N_, directly or with `M100`
- _Tool specific:_ Optional

    Function |  Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Output (no tool) | `M100 ({out1:t})` | Output | Output as `out1`: `{do1fn:1}`, No tool: `{do1tn:0}` | _None_
    Output (part of tool) | `M6 T3` -> `M100 ({out2:t})` | Output |  Output as `out2`: `{do1fn:2}`, Tool 3: `{do1tn:3}` | _None_

### System Functions

#### Feedhold Input

#### Limit Switch Input

#### Homing Switch Input

#### Shutdown Input

#### Interlock Input

### Spindle Functions
  - Controlled by `M3`, `M4`, and `M5`
  - May be affected by feed hold behaviors and other functions
  - Also may be affected by `M6` in machines with more than one spindle - i.e. changing tool number may also change spindle
  - _Tool specific:_ Y

    Function | Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Speed | `M3 Snnn` `M4 Snnn` | Output (w/PWM) | Other function: `{do1fn:0}`, Tool 1: `{do1tn:1}` | Tool 1 spindle "speed" is output 1: `{t1sps:1}`
    On/Off | `M3`, `M4`, `M5` | Output |  Other function: `{do2fn:0}`, Tool 1: `{do2tn:1}` | Tool 1 spindle "on" is output 2: `{t1spon:2}`
    Direction | `M3` or `M4` | Output |  Other function: `{do3fn:0}`, Tool 1: `{do3tn:1}` | Tool 1 spindle "direction" is output 3: `{t1sps:3}`

### Heater Functions
  - Controlled by JSON as `out`_N_, directly or with `M100`
  - `M101` waits can wait for the `at` subvalue of a `he` object, like this: `M101 ({he1at:t})`
  - _Tool specific:_ Optional

    Function |  Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Heater | `{he1st:100}` (etc.) | Output | Output 1 as 'other': `{do1fn:0}` | Assign heater to output 1: `{he1out:1}`
    Temperature Sensor | (same) | Temperature Sensor | Analog input 2 as 'other': `{ai2fn:0}` | Assign `ts4` to use `ai2`: `{ts4in:2}`, , Assign `ts4` function to `other`: `{ts4fn:0}`, Assign heater temp. sensor to `ts4`: `{he1ts:4}`
    Fan |  (same) | Output | Output 3 as 'other': `{do3fn:0}` | Assign heater temp. sensor to `out3`: `{he1fo:3}`

### Temperature Sensor Functions
  - Can be read like an analog input with JSON as `ts`_N_, but the floating point value is in degrees C.
  - _Tool specific:_ No, can be attached _to_ by a tool, however.

    Function |  Control via | Type | DI/DO/AI Setting | Other Setting
    --- | --- | --- | --- | ---
    Input | `{temp1:n}` | Input (Bridge) | Analog input 4 as 'other': `{ai4fn:0}` | Assign temp. sensor 2 to analog input 4: `{ts2in:4}`, Assign temp. sensor 2 function to `temp1`: `{ts2fn:1}`


**Detailed Controls for the above functions (incomplete)**:

 Name (spaces added) | Description | R/W | Values
 ---- | ---------- | :-: | -------
 `t1` | "tool 1" group | RO | Tools start from 1 and go up
 `t1 sp` | "tool 1 spindle" group | RO | Tool 1 spindle group
 `t1 sp s` | "tool one spindle speed output" | RW | Output number of a PWM-capable output to control spindle speed controlled by `M3`, `M4`, and `M5`.
 `t1 sp csl` | "clockwise speed low" | RW | Minimum clockwise speed (matching the `S` word of an `M3`)
 `t1 sp cpl` | "clockwise phase low" | RW | Output value (from `0.0` to `1.0`) representing to duty cycle at min. speed
 `t1 sp csh` | "clockwise speed high" | RW | Maximum clockwise speed (matching the `S` word of an `M3`)
 `t1 sp cph` | "clockwise phase high" | RW | Output value (from `0.0` to `1.0`) representing to duty cycle at max. speed
 `t1 sp wsl` | "counter-clockwise speed low" | RW | Minimum counter-clockwise speed (matching the `S` word of an `M3`)
 `t1 sp wpl` | "counter-clockwise phase low" | RW | Output value (from `0.0` to `1.0`) representing to duty cycle at min. speed
 `t1 sp wsh` | "counter-clockwise speed high" | RW | Maximum counter-clockwise speed (matching the `S` word of an `M4`)
 `t1 sp wph` | "counter-clockwise phase high" | RW | Output value (from `0.0` to `1.0`) representing to duty cycle at max. speed
 `t1 sp of` | "phase when off" | RW | Output value (from `0.0` to `1.0`) representing to duty cycle when the spindle is off   
 `t1 sp on` | "tool 1 on/off output" | RW | Output number of a digital output that control spindle ON/OFF (but not speed). Active being ON. Set active high/low on the output (`do`) directly.
 `t1 sp dir` | "tool 1 direction output" | RW | Output number of a digital output that control spindle direction. Active being clockwise. Set active high/low on the output (`do`) directly.
 `t1 out1` | "tool 1 output 1 value" | RO | Tool 1 general output 1 group. See `out1` for subkeys.
 `t1 in1` | "tool 1 input value" | RO | Tool 1 general digital input 1 group. See `in1` for subkeys.
 `t1 ain1` | "tool 1 analog input value" | RO | Tool 1 general analog input 1 group. See `ain1` for subkeys.
 `t1 he1` | "tool 1 heater 1" group | RO | Tool 1 heater 1 group. See `he1` for subkeys.
 `t1 fan1` | "tool 1 fan 1" group | RO | Tool 1 fan 1 group. See `fan` for subkeys.
 `out1` | "output 1 value" | RW | Control the value of a given output. Read returns the last set value.
 `in1` | "input 1 value" | RO | Access the value of a given input. Return value is BOOL True or False.
 `ain1` | "analog input 1 value" | RO | Access the value of a given analog input. Return value is float between `0` and `1`.
 `he1` | "heater 1" group | RO | Heater 1
 ... | TODO | TODO | TODO
 `fan1` | "heater 1" group | RO | Fan 1
 ... | TODO | TODO | TODO
 `ts1` | "temperature sensor 1" group | RO | Temperature sensor 1
 ... | TODO | TODO | TODO
 `temp1` | "temperature value 1" | RO | Temperature output 1
 ... | TODO | TODO | TODO