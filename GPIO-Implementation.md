This page discusses upcoming functionality being added to the GPIO system.<br>

### Pages:
- [GPIO Overview](GPIO-Design-Discussion)
- [GPIO Design Goals](gpio-design-goals)
- [GPIO Primitives](gpio-primitives)
- [GPIO Objects and Bindings](gpio-objects-and-binding)
- GPIO Implementation

See also:
- [G2 GPIO System](Digital-IO)
- [JSON Operation](JSON-Operation)
- [Toolheads](Toolheads)

# GPIO Implementation

GPIO implementation is discussed in the following sections:

1) [Consuming an input or driving an output](#consuming-an-input-or-driving-an-output)
2) [Base Classes: How inputs/outputs are implemented](#base-classes---how-io-is-implemented)
3) [How to implement a new input/output subclass](#how-to-implement-a-new-inputoutput)

All GPIO primitives have two programmatic interfaces:

- JSON/text mode
  - This interface consists of a series of functions that take `nvObj_t *` inputs and have a return of `stat_t`
- Direct
  - These are functions that will be used by the rest of the system and may have different inputs and outputs
  - The direct functions are used by the JSON/text mode functions

# Consuming an input or driving an output

Here we will focus on the direct interface, which is what will be used from other parts of the code (the objects that are binding to the primitives).

## Digital Inputs

In `gpio.{h,cpp}` the digital inputs are defined (roughly) as:

```c++
gpioDigitalInput*   const d_in[D_IN_CHANNELS];
```

The `d_in` pointers will be subclasses of `gpioDigitalInput` which has the following interface:

- `bool getState()`
  - _Returns input state: `true` means input is `active` state, accounting for polarity._
  - Example:
    ```c++
    if (d_in[input_num]->getState()) {
      // do something only if that input is active
    }
    ```
- `inputAction getAction()`
  - _Returns the `inputAction` enum value for the currently set action_

- `bool setAction(const inputAction)`
  - Sets the action for the input to the given value
  - _Returns `false` if the action could not be set, `true` if it was_

- `ioEnabled getEnabled()`
  - _Returns the `ioEnabled` enum value of this input_
  - Keep in mind that both `IO_UNAVAILABLE` and `IO_DISABLED` mean "disabled," but only `IO_DISABLED` means that `setEnabled(...)` may be used
  - Example:
    ```c++
    if (d_in[input_num]->getEnabled() != IO_ENABLED) {
      // this pin is not available
    }
    ```

- `bool setEnabled(const ioEnabled)`
  - Sets the enabled state for the input to the given value - only `IO_ENABLED` and `IO_DISABLED` are valid
  - _Returns `false` if the enabled state could not be set, `true` if it was_

- `ioPolarity getPolarity()`
  - _Returns the current `ioPolarity` of this pin_
  - *Note:* It is advised that code **only** use the enum values `IO_ACTIVE_HIGH` (non-inverted polarity) and `IO_ACTIVE_LOW` (inverted polarity) and never their numeric values internally

- `bool setPolarity(const ioPolarity)`
  - Sets the polarity for the input to the given value - either `IO_ACTIVE_HIGH` (non-inverted polarity) or `IO_ACTIVE_LOW` (inverted polarity)
  - _Returns `false` if the polarity could not be set, `true` if it was_

- `const uint8_t getExternalNumber()`
  - _Returns the current external number of this pin_
  - _See next function for more info_

- `bool setExternalNumber(const uint8_t)`
  - Sets the external number for the input to the given value
    - `0` means no external number is assigned
    - If the value is anything but `0` then the JSON object `in`_N_ is exposed, where _N_ is the provided value
  - _Returns `false` if the external number could not be set, `true` if it was_

### Digital Input Event Handler

The `gpioDigitalInputHandler` class is used to create handlers for input events. It can be used stand-alone, as a parameter of a class, or as a superclass. Here we will only demonstrate using it as a stand-alone object.

There are only three values in a `gpioDigitalInputHandler`, and they are (in this order, which is vital):

- `callback` - this `const` is the function with the signature `bool(const bool state, const inputEdgeFlag edge, const uint8_t triggering_pin_number)`
  - `state` is the current state of the input where `true` is `active`, accounting for polarity.
  - `edge` is either `INPUT_EDGE_LEADING` (transition from inactive to active) or `INPUT_EDGE_TRAILING` (transition from active to inactive)
  - `triggering_pin_number` is the number of the pin, so for `din3` that would be `3`
  - The callback must return `true` if no other handlers are to see the event, or `false` to allow this event to propagate 
  - **IMPORTANT** the callback is called from an interrupt context, and must act accordingly so as to not interrupt the typical flow of the system! The callback should be relatively light weight and must be non-blocking (run-to-completion). In general, requesting processing (e.g. feedholds), setting flags or altering state machines for further processing from less critical contexts should be what's done here.
- `priority` - this `const` is the priority of the event handler, with `100` being the highest and `1` being the lowest. By convention, use 5 for "normal" handlers, and 100 for special handlers like homing and probing that need to consume events w/o firing lower-priority handlers.
  - > This has not been used yet to decide how we should group these
- `next` - this is the only mutable value, and should always be initialized as `nullptr`

You set the parameters during the initialization of the object, and the callback is generally a lambda. An example:

```c++
gpioDigitalInputHandler limitHandler {
    [&](const bool state, const inputEdgeFlag edge, const uint8_t triggering_pin_number) {
        if (edge != INPUT_EDGE_LEADING) { return; }
        limit_requested = true; // record that a limit was requested for later processing
        return false; // allow lower-priority handlers to see this event
                      // return true to consume the event and stop further processing 
    },
    5,      // priority (for example)
    nullptr // next - set to nullptr to start with
};
```

Now that the object is created, it must be _registered_. Registration is done via the global `gpioDigitalInputHandlerList din_handlers[]` array. Each `gpioDigitalInputHandlerList` is a linked list for a different `inputAction` value, with the special value `INPUT_ACTION_INTERNAL` being used for **all** pin changes.

An example of how to register the `limitHandler` we instantiated above:

```c++
din_handlers[INPUT_ACTION_LIMIT].registerHandler(limitHandler);
``` 

And to remove it later:

```c++
din_handlers[INPUT_ACTION_LIMIT].deregisterHandler(limitHandler);
```

-----------------------------------------------------------------------------

## Digital Outputs

In `gpio.{h,cpp}` the digital outputs are defined (roughly) as:

```c++
gpioDigitalOutput*  const d_out[D_OUT_CHANNELS];
```

The `d_out` pointers will be subclasses of `gpioDigitalOutput` which has the following interface:

- `float getValue()`
  - _Returns the current (last set) value of this output_
  - Value is a `float` from `0.0` (fully inactive) to `1.0` (fully active), where polarity is accounted for

- `bool setValue(const float)`
  - Sets the PWM duty cycle of the output base on the provided `float` with a value from `0.0` to `1.0`
  - _Returns `false` if the value could not be set, `true` if it was_

- `float getFrequency()`
  - _Returns the last set frequency value of this output, or `0` if it was never set_

- `bool setFrequency(const float)`
  - Set the frequency (in Hz) of the PWM of this pin, if it is capable of PWM
  - _Returns `false` if the frequency could not be set, `true` if it was_

- `ioEnabled getEnabled()`
  - _Returns the `ioEnabled` enum value of this input_
  - Keep in mind that both `IO_UNAVAILABLE` and `IO_DISABLED` mean "disabled," but only `IO_DISABLED` means that `setEnabled(...)` may be used
  - Example:
    ```c++
    if (d_out[output_num]->getEnabled() < IO_ENABLED) {
      // this pin is not available
    }
    ```

- `bool setEnabled(const ioEnabled)`
  - Sets the enabled state for the input to the given value - only `IO_ENABLED` and `IO_DISABLED` are valid
  - _Returns `false` if the enabled state could not be set, `true` if it was_

- `ioPolarity getPolarity()`
  - _Returns the current `ioPolarity` of this output_
  - *Note:* It is advised that code **only** use the enum values `IO_ACTIVE_HIGH` and `IO_ACTIVE_LOW` and never their numeric values internally

- `bool setPolarity(const ioPolarity)`
  - Sets the polarity for the output to the given value - either `IO_ACTIVE_HIGH` or `IO_ACTIVE_LOW`
  - _Returns `false` if the polarity could not be set, `true` if it was_

- `const uint8_t getExternalNumber()`
  - _Returns the current external number of this pin_
  - _See next function for more info_

- `bool setExternalNumber(const uint8_t)`
  - Sets the external number for the input to the given value
    - `0` means no external number is assigned
    - If the value is anything but `0` then the JSON object `in`_N_ is exposed, where _N_ is the provided value
  - _Returns `false` if the external number could not be set, `true` if it was_


## Analog Inputs

In `board_gpio.{h,cpp}` the digital outputs are defined (roughly) as:

```c++
gpioAnalogInput*    const a_in[A_IN_CHANNELS];
```

The `a_in` pointers will be subclasses of `gpioAnalogInput` which has the following interface:

- `float getValue()`
  - _Returns the last computed voltage value of this input_
  - Value is generally is from `0.0` to the max voltage the ADC can read, which is `3.3` for all currently supported boards
  - For non-ADC (external) analog inputs this value may be negative and potentially have values outside of this range

- `float getResistance()`
  - _Returns the last computed resistance value of this input, based on the computed circuit_
  - If the circuit is not configured, it will return -1
  - The range and value is determined by the circuit

- `AnalogInputType_t getType()`
  - Gets the `AnalogInputType_t` enum value of this input
  - See above documentation for more info

- `bool setType(const AnalogInputType_t)`
  - _Returns `false` if the frequency could not be set, `true` if it was_

- `AnalogCircuit_t getCircuit()`
  - Gets the `AnalogCircuit_t` enum value of this input
  - See above documentation for more info

- `bool setCircuit(const AnalogCircuit_t)`
  - _Returns `false` if the frequency could not be set, `true` if it was_

- `float getParameter(const uint8_t p)`
  - Returns the float value of tha parameter number specified by `p`
  - See above documentation for more info

- `bool setParameter(const uint8_t p, const float v)`
  - Sets the value of the parameter number specified by `p` to the float value `v`
  - _Returns `false` if the frequency could not be set, `true` if it was_

- `void startSampling()`
  - Starts one sample
  - Should be able to be called during an in-progress sample and be ignored
  - May be ignored (but must be implemented) by some analog input types if sampling does not require polling

- > Add `getEnabled()` and `setEnabled()` ?

<!--
- `ioEnabled getEnabled()`
  - _Returns the `ioEnabled` enum value of this input_
  - Keep in mind that both `IO_UNAVAILABLE` and `IO_DISABLED` mean "disabled," but only `IO_DISABLED` means that `setEnabled(...)` may be used
  - Example:
    ```c++
    if (d_out[output_num]->getEnabled() < IO_ENABLED) {
      // this pin is not available
    }
    ```

- `bool setEnabled(const ioEnabled)`
  - Sets the enabled state for the input to the given value - only `IO_ENABLED` and `IO_DISABLED` are valid
  - _Returns `false` if the enabled state could not be set, `true` if it was_
-->
** Not yet implemented: **

- `const uint8_t getExternalNumber()`
  - _Returns the current external number of this pin_
  - _See next function for more info_

- `bool setExternalNumber(const uint8_t)`
  - Sets the external number for the input to the given value
    - `0` means no external number is assigned
    - If the value is anything but `0` then the JSON object `in`_N_ is exposed, where _N_ is the provided value
  - _Returns `false` if the external number could not be set, `true` if it was_

-----------------------------------------------------------------------------

# Base Classes - How IO is implemented

For digital inputs the parent class is `gpioDigitalInput` and for outputs the parent class is `gpioDigitalOutput`. These each provide both the JSON interface that calls the direct interface as well as pure virtual (`virtual` functions with no provided definition) functions for the direct interface. Subclasses only need to implement and override those virtual functions of the direct interface.

For input, there are provided concrete subclasses for [`Motate::IRQPin`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L310-L326)-compliant objects `gpioDigitalInputPin<typename Pin_t>`. These also handle the pin change interrupts (using the `IRQPin` callback mechanism) and `gpioDigitalInputHandler` calls. _Note: The `typename Pin_t` does **not** need to be a subclass of `Motate::IRQPin` but only must adhere to the minimal interface used (see below)._

For analog inputs there are provided concrete subclasses for [`Motate::ADCPin`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L400-L785)-compliant objects `gpioAnalogInputPin<typename ADCPin_t>`. These also handle the analog update interrupts (using the `ADCPin` callback mechanism). There is not currently a mechanism to subscribe to these updates.

Similarly, for output pins there are provided concrete subclasses for [`Motate::PWMOutputPin`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L792-L919)-compliant objects `gpioDigitalOutputPin<typename Pin_t>`.

All these are used (and thus linked to actual pins or fake pins) in the `board_gpio.{h,cpp}` files.

-----------------------------------------------------------------------------

# How to implement a new input/output

It's possible to create new digital input or output types by either subclassing `gpioDigitalInput`, `gpioAnalogInput`, or `gpioDigitaOutput` _or_ by implementing the appropriate Motate pin interface methods. In either case, you create those objects in `board_gpio.{h,cpp}` and place them in the appropriate array of pointers (`d_in[]`, `a_in[]`, or `d_out[]`).

The interface used by `gpioDigitalInputPin<typename Pin_t>` of a `Pin_t` (typically `Motate::IRQPin`) are:

| Function | Notes |
| --- | --- |
[`IRQPin(const PinOptions_t options, const std::function<void(void)> &&interrupt, ...)`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L317) | This is the constructor, the name would change accordingly of course <br/> `interrupt` must be called when a pin changes, and that is expected to be called from an interrupt context (but might not be) <br/> `...` indicates that additional parameters passed to the contructor of `gpioDigitalInputPin<>` will be forwarded
`bool isNull()` | This should always return the same value, and unless the pin is physically unavailable, it should return `false`
`void setOptions(const PinOptions_t options, const bool fromConstructor=false)` | `PinOptions_t` is defined as `((polarity == IO_ACTIVE_LOW) ? kPullUp\|kDebounce : kDebounce)` and `fromConstructor` will only be true the first time it's called
`operator bool()` | this returns the actual boolean value of the "pin" and does **not** account for polarity


The interface used by `gpioDigitalOutputPin<typename Pin_t>` of a `Pin_t` (typically `Motate::PWMOutputPin` or `Motate::PWMLikeOutputPin`) are:

| Function | Notes |
| --- | --- |
[`PWMOutputPin(const PinOptions_t options, ...) `](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L915) | This is the constructor, the name would change accordingly of course <br/> `options` is set as `((polarity == IO_ACTIVE_LOW) ? kStartHigh\|kPWMPinInverted : kStartLow)` <br/> `...` indicates that additional parameters passed to the contructor of `gpioDigitalOutputPin<>` will be forwarded
`bool isNull()` | This should always return the same value, and unless the pin is physically unavailable, it should return `false`
`void setOptions(const PinOptions_t options, const bool fromConstructor=false)` | `PinOptions_t` is defined as `(polarity == IO_ACTIVE_LOW) ? kStartHigh\|kPWMPinInverted : kStartLow)` - note that checking against `Motate::kPWMPinInverted` will indicate you need to invert the output <br/> `fromConstructor` will only be true the first time it's called
`void setFrequency(const uint32_t freq)` | requests the pwm frequency provided
`void operator=(const float value)` | used to set the output value, passed as a float from `0.0` to `1.0` and is **not** adjusted for polarity (use `setOptions()` to determine if output needs inverted)

The interface used by `gpioAnalogInputPin<typename ADCPin_t>` of a `ADCPin_t` (typically `Motate::ADCPin`) are:

| Function | Notes |
| --- | --- |
[`ADCPin(const PinOptions_t options, std::function<void(void)> &&interrupt, ...)`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L529-L532) | This is the constructor, the name would change accordingly of course <br/> `interrupt` must be called when a pin changes, and that is expected to be called from an interrupt context (but might not be) <br/> `...` indicates that additional parameters passed to the contructor of `gpioAnalogInputPin<>` will be forwarded
`bool isNull()` | This should always return the same value, and unless the pin is physically unavailable, it should return `false`
[`void setVoltageRange(const float vref, const float min_expected = 0, const float max_expected = -1, const float ideal_steps = 1)`](https://github.com/synthetos/Motate/blob/sams70-port/MotateProject/motate/MotatePins.h#L572-L578) | Sets what is expected as an valid input range and requested resolution so offsetting and scaling may be used if available (can be safely ignored, but must be implemented)
`void setInterrupts(const uint32_t interrupts)` | Always called as `pin.setInterrupts(Motate::kPinInterruptOnChange|Motate::kInterruptPriorityLow);`
`float getTopVoltage()` | returns the maximum possible value
`void startSampling()` | start the sampling (if necessary) - when the new value is in `interrupt` (given in the constructor) must be called
`int32_t getRaw()` | return the raw ADC value (used only for debugging, and may not be called at all)
`float getVoltage()` | returns the ADC value converted to a voltage level but otherwise unprocessed
`static constexpr bool is_differential` | a static constant that must be in the class - `true` means the value is treated as a differential and the valid range is `-getTopVoltage()` to `getTopVoltage()`, and `false` means the valid range is `0.0` to `getTopVoltage()`
***
