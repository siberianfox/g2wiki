This page discusses upcoming functionality being added to the GPIO system.<br>

### Pages:
- [GPIO Overview](GPIO-Design-Discussion)
- [GPIO Design Goals](gpio-design-goals)
- GPIO Primitives
- [GPIO Objects and Bindings](gpio-objects-and-binding)
- [GPIO Implementation](gpio-implementation)

See also:
- [G2 GPIO System](https://github.com/synthetos/g2/wiki/Digital-IO-(GPIO))
- [JSON Operation](https://github.com/synthetos/g2/wiki/JSON-Operation)
- [Toolheads](https://github.com/synthetos/g2_private/wiki/Toolheads)

# GPIO Primitives
The three IO primitives are **digital input**, **analog input**, and **digital output**. Analog output is not implemented, but practically speaking we always use pulse width modulation (PWM) for variable outputs, so PWM functionality is folded into the digital outputs and digital outputs can be driven using analog values (0.000 - 1.000).

IO Primitives are used to configure IO, but not to read or set it. The value of the primitive is accessible based on its exposed-as function. The primitive may also be bound to objects and object member functions that map (bind) to the IO primitives.

The IO primitives are exposed as per these examples:
- _`diN`_ is exposed as _`inM`_ and read as `{"in1":null}` (shorthand: `{"in1":n}` or `{in1:n}`)
- _`aiN`_ is exposed as _`ainM`_ and read as `{ain1v:n}`
- _`doN`_ is exposed as _`outM`_ and read as `{out1:n}` and written as `{out1:1}`

_Note: In some cases an object binding will "unexpose" the IO. I.e. the IO is no longer accessible through its exposed-as setting_

## Settings Common to all IO Types

### {...en:n} Enable Setting

Each IO primitive has a configurable "enabled" setting, with a subkey of `en`:

   Enabled | State | Description
   ------|-------|-----
   -1 | Unavailable | The IO is *not* available and cannot be enabled - usually because it does not physically exist in this hardware configuration. Set at compile time.
   0 | Disabled | The IO is available in the system but has been disabled
   1 | Enabled | The IO is available in the system and is enabled

**Examples**:
- `{di2en:1}` - enable digital input pin 2
- `{ai3en:0}` - disable analog input pin 3
- `{ai3en:n}` --> `{ai3en:0}` - read analog input pin 3 and find that it's disabled
- `{do16en:n}` --> `{do16en:-1}` - read digital output pin 16 and find that it cannot be enabled (possibly because it does not exist on this hardware)

**Notes**:
- **The `en` setting replaces the mode (`mo`) in g2core versions prior to `10_.__` (TBD). The `mo` subkey is no longer defined and will return an error if accessed**
- The pin enable value can be queried and will return one of 1, 0 or -1
- A pin with a `en` value of `-1` is read-only - you cannot enable the pin nor use it.
- Note that attempting to perform any other operation on a disabled or unavailable IO primitive will return NULL and an error - see the Primitive descriptions for details 

### {...po:n} Polarity Setting
#### !!! PLEASE READ !!!
#### YOU MUST MAKE THIS ONE-TIME CHANGE TO YOUR CONFIGURATIONS!!!

The digital IO primitives (not including analog input) have a configurable polarity setting:

   Polarity | State | Description
   ------|-------|-----
   0 | Active_High (Non-Inverted) | The pin being `hi` reports as `1`. Corresponds typical wiring for normally-closed switches
   1 | Active_Low (Inverted) | The pin being `hi` reports as `0`. Corresponds to typical wiring for normally-open switches

**Examples**:
- `{di2po:0}` - set digital input pin 2 to non-inverted polarity 
- `{do3po:1}` - set digital output pin 3 to inverted polarity
- `{do3po:n}` --> `{do3en:0}` - return polarity setting for digital output pin 3

**Note**: The above definitions of Normal and Inverted are the opposite of the way DI's currently work in g2core versions prior to `102.xx` builds, which is `0=Active_low` and `1=Active_high`. This change makes the Enable and Polarity definitions consistent across all IO types, and also with motor polarity settings.

### Virtual Inputs and Outputs
Virtual inputs and outputs are merely signals that are produced or consumed by program functions instead of by a physical IO pin. An ADC is a good example. The ADC value is heavily conditioned before being exposed as an analog signal. Another example would be a serial-connected spindle that sends a message when the spindle is at RPM. An object can read that message and assert an "at-speed" signal.

-----------------------------------------------------------------------------------------------------

# Digital Inputs
A digital input "pin" may be a physical pin or the output from an internal signal such as a timer timeout or an object that outputs 1 or 0 (true or false). Digital inputs may be bound to an internal object to act as in input source, and may be bound to an _Exposed_As_ object to expose the value.

In addition to any assigned object(s), inputs may have an _Action_ that occurs immediately, i.e. the action is tied to the actual pin firing interrupt. This is needed for some time-sensitive operations like machine halts or catastrophic failures (e.g. Panic).

## Digital Input Properties

Exposed-As _`inM`_:

- The input may be assigned to an _`inM`_ exposed via JSON
- The digital input value reads as `0` (off) or `1` (on)
- The value is sense-corrected according to the Polarity (`po`) setting - Non-inverted (`1` = high) or Inverted (`1` = low)
- The value is conditioned - it's deglitched, debounced or otherwise conditioned
- The exposed value is read-only - _`inM`_ cannot be written and attempted writing is an error
- Attempting to read a value of a disabled (`{in1en:0}`) (or unavailable `{in1en:-1}`) `in` will return a `null` value and the response footer will indicate an error.
- Digital inputs can be waited on using the `M101` command, thus a Gcode file can be paused until an input condition specified in the M101 is met. An example is waiting on a heater to achieve its set temperature.

## Digital Input Configuration Values

Input pins _`diN`_ can be configured in JSON:

  Name | Description | Values
  ------|------------|---------
  `{di1en:` | enabled | -1=unavailable, 0=disabled, 1=enabled
  `{di1po:` | polarity | 0=non-inverted (active high), 1=inverted (active low)
  `{di1ac:` | action | 0=none, 1=stop, 2=fast_stop... see table below
  `{di1in:` | exposed-as | 0=not-exposed, 1-M = bound to `in1` through `inM`

Inputs can be exposed in JSON:

  Name | Description | Values
  ------|------------|---------
  `{inM:n}` | read value | returns the state of the input

Actions that can be invoked immediate on digital inputs:

  Action | Value | Description
  -----|-------|------------
  `None`        | `0` | No action taken
  `Stop`        | `1` | Request controlled feedhold that preserves position
  `Fast Stop`   | `2` | Request controlled feedhold at high jerk that preserves position
  `Halt`        | `3` | Cease steps immediately and does not guarantee preserved position
  `Cycle Start` | `4` | Start or resume motion after a feedhold or stop
  `Alarm`       | `5` | Place system in an alarm state (and may initiate further system actions)
  `Shutdown`    | `6` | Initiate system shutdown
  `Panic`       | `7` | Initiate system panic
  `Reset`       | `8` | Perform hard reset of the controller
  **Temporary:** | |
  `Limit`       | `9` | Initiate limit processing (Q: _Put under `lim` object?_)
  `interlock`   | `10` | Initiate interlock processing (Q: _Put under `saf` object?_)

**Notes:**

- Enabled 
  - Enabled will return `-1` if an input is not available due to hardware or configuration. In this state, the `di` object is read-only, and the other keys will return `null`.
- Exposed As:
  - A digital input can only be exposed_as a single `in` function (multiple inN's for a single input are not supported)
  - `M` is limited to 16 and will return a range error if exceeded
- Attempting to configure a non-existent `di` will return a `null` value and an error status code

-----------------------------------------------------------------------------------------------------

# Digital Outputs
A digital output "pin" may be a physical pin or the output may be an internal signal.

## Digital Output Properties

All digital outputs share the same JSON interface if they are simple binary `on-off` outputs or if they are PWM-capable outputs.

- The output may be exposed-as _`outM`_ to be visible via JSON
- Output values may be written as integers: `0` and `1` (inactive, active)
- Output values may be written as booleans: `f` and `t` (converted to 0,1 internally)
- Output values may be written as floats: `0.000` and `1.000` (inactive, active)
  - A floating point value will set the duty cycle of PWM capable pins
  - For pins that are not PWM capable, the value will be interpreted as `(bool)(value >= 0.5)`
  - ...anything less than 0.5 will read as inactive, >= 0.5 as active
- The actual output is polarity corrected based on the polarity setting 
  - If polarity is Non-inverted the pin value is the same as the written value: e.g. writing 1 will set the pin HI. If polarity is Inverted the output pin is opposite the written value. 
  - The exposed value reads back the value it was set to as a float, which may not be the value provided. For example, if a non-PWM-capable output was set to `0.75` it will read back as `1.000`
  - Set and read values will always align, regardless of polarity. IOW, if polarity is Inverted setting the output to `1` will read back as `1` to indicate "active", reflecting the value set, even though the pin output is LOW
- Attempting to read or write a value of a disabled or unavailable `out` will return a `null` value. A status code may also return an error if the request was an unavailable or non-existent output.
- For PWM-capable pins, the duty cycle is relative to "active", so a value of `0.75` means "active" 75% of the time. With polarity set to non-inverted (`{do1po:0}`) this means HIGH 75% of the time, and with polarity set to inverted (`{do1po:1}`) it means LOW 75% of the time.

## Digital Output Configuration Values

Output pins can be configured in JSON via _`doN`_ as below:

  Name | Description | Values
  ------|------------|---------
  `{do1en:` | enabled | -1=unavailable, 0=disabled, 1=enabled
  `{do1po:` | polarity | 0=non-inverted (active high), 1=inverted (active low)
  `{do1out:` | exposed-as | 0=not-exposed, 1-M = bound to `out1` through `outM`
  `{do1frq:` | frequency | -1=PWM unavailable, 0=PWM off, >0 = PWM frequency in Hz

Outputs can be exposed in JSON:

  Name | Description | Values
  ------|------------|---------
  `{outM:n}` | set/read value | sets or returns the state of the output

Notes:
- Enabled 
  - Enabled will return `-1` if an input is not available due to hardware or configuration. In this state, the `do` object is read-only, and the other keys will return `null`.
- Exposed-As
  - A digital output can only be exposed as a single `out` function (multiple expose points are not supported)
  - `M` is limited to 16 and will return a range error if exceeded
- Frequency
  - Can be set to any supported value but may be limited to a system-wide max
  - PWM channels that share frequencies (common timers) may also be limited
    - Due to hardware limitations, changing the frequency of one output may change the frequency of other outputs
  - If the IO does not support PWM `frq` will return `-1` when read, and be read-only
- Deprecated Values
  - The PWM configuration stated above does not have a few of the settings currently supported in v8 and the PWM object of g2. Duty Cycle settings present in earlier revisions have been removed from the `do` object, to be placed on the object driving the output - for example the toolhead object for the spindle type toolhead, or in any other case where there is something else driving the output. Omitted settings include:
    - Duty cycle setting
    - PWM OFF level - used for RC power controllers. Setting the right OFF value should be the responsibility of the toolhead functions
    - Asymmetric CW and CCW settings (independent). We may not need these. Never have so far.

-----------------------------------------------------------------------------------------------------

# Analog Inputs
An analog input "pin" may be a physical pin or an internal signal such as a function the returns a floating point value.

## Analog Input Properties

Exposed-As _`ainM`_
- The input may be assigned to an _`ainM`_ generic function to be exposed via JSON
- The analog input value reads as either a raw voltage or as a computed "resistance" value. Other objects may then further compute from there to a final value.
- The value is conditioned - it's sampled and averaged or may have other signal processing applied
- The exposed value is read-only - _`ainM`_ cannot be written and will return an error if attempted
- Attempting to read a value of a disabled (or unavailable) `ain` will return a `null` value. An error status code may also be returned if the request was an unavailable or non-existent input
- Analog inputs cannot _directly_ be waited on by `M101`, but components may have "wait" functions that can be used for synchronization to analog inputs. These component functions may be associated with an analog pin, and then indirectly provide binary values than can be waited on. For example, a heater may use an analog input for the temperature sensor, and then a wait can be used for when the heater is at temperature - example `he1at:1`.

## Analog Input Configuration Values

Input pins can be configured in JSON via _`aiN`_ as below:

  Name | Description | Values
  ------|------------|---------
  `{ai1en:` | enabled | -1=unavailable, 0=disabled, 1=enabled
  `{ai1ain:` | exposed-as | 0=not-exposed, 1-M = bound to `ain1` through `ainM`
  `{ai1ty:` | type (read-only) | 0=single-ended, 1=differential, 2=external
  `{ai1ct:` | circuit | 0=disabled, 1=simple pull-up, 2=raw input (external), 3=inverted op-amp
  `{ai1p1:` | parameter 1 | value interpretation determined by circuit
  `{ai1p2:` | parameter 2 | value interpretation determined by circuit
  `{ai1p3:` | parameter 3 | value interpretation determined by circuit
  `{ai1p4:` | parameter 4 | value interpretation determined by circuit
  `{ai1p5:` | parameter 5 | value interpretation determined by circuit

Analog inputs can be exposed in JSON:

  Name | Description | Values
  ------|------------|---------
  `{aiMain:n}` | exposed-as |  0=not-exposed, 1-M = bound to `ain1` through `ainM`
  `{ainMvv:n}` | read voltage value | reads the value of the input in actual voltage, nominal range is 0 - VCC
  `{ainMrv:n}` | read resistance value | reads the resistance of the input based on the circuit and parameters

Notes:

- Enabled - Enabled will return `-1` if an analog input is not available due to hardware or configuration. In this state, the `ai` object is read-only, and the other keys may return `null` or nonsense values.
- Exposed-As
  - An analog input can only bind to a single `ain` function (multiple exposes are not supported)
  - `M` is limited to 8 (currently) and will return a range error if exceeded
  - **Note** that unlike digital input and outputs, there are three subkeys of the analog input exposed key (_`ainM`_)
- Attempting to configure a non-existent `ai` will return a `null` value and an error status code
