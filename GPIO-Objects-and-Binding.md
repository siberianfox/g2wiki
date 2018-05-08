This page discusses upcoming functionality being added to the GPIO system.<br>

### Pages:
- [GPIO Overview](GPIO-Design-Discussion)
- [GPIO Design Goals](gpio-design-goals)
- [GPIO Primitives](gpio-primitives)
- GPIO Objects and Bindings
- [GPIO Implementation](gpio-implementation)

See also:
- [G2 GPIO System](Digital-IO)
- [JSON Operation](JSON-Operation)
- [Toolheads](Toolheads)

# GPIO Objects and Binding

## Objects

We use the term 'object' very loosely here. An object is any g2 code that performs some function, like raise an alarm if a limit is hit, or detect that a heater has reached a stable set temperature.

An IO may be associated with a object and be accessed as a member function of that object. For example:
- An input assigned to an object like safety interlock input `saf` can be read as `{safes:n}` and assigned to follow digital input 4 with `{safin:4}`
- An output assigned to a object like mist coolant `com` can be written as `{comen:1}`

Not all objects expose the raw value of an input or output. For example, when an output is assigned to the "spindle speed" function there is no way to read or write the raw PWM duty-cycle output value. It can only be set via Gcode `S` words and read via the JSON `{sps:N}` command (or text-mode equivalent).

## Bindings
Bindings are how we connect IO to signal sources and useful functions. Bindings are configurable and allow a machines to be customized for specific applications. 

- *Binding* connects an IO primitive to one or more objects or object member functions
  - Example: `xhi:N` (X Homing Input) tells the X axis object to use digital input N for homing
  - Example: `he1at:n` (Heater 1 At Temperature) is a digital signal generator that triggers when heater has reached set temperature. It may be bound to a pin or other function.
- Bindings are configurable at run time
  - IO primitives may be bound and rebound during machine or job setup; this is not recommended during job processing
  - Some bindings may be hard-wired at compile time for some highly complex functions and thus not be re-configurable
- Bindings can specify one or more of the following trigger events
  - Trigger on active - trigger for each "pass" through the active state (rarely used)
  - Trigger on inactive  - trigger for each "pass" through the inactive state (rarely used)
  - Trigger on leading edge - trigger once on transition from inactive to active state
  - Trigger on trailing edge - trigger once on transition from active to inactive state
- Bindings are non-modal, but objects might not be
  - Bindings are in effect when the are set, and are in effect regardless of machine state
  - However, objects may interpret the bound signals modally, like homing and probing do. For example, the homing input binding is used during a homing operation and may supersede a limit object bound to that same input. Once the homing cycle is complete the limit function is restored. This logic exists in the code, not in the bindings, which are static.
- Multiple functions may be bound to an IO, and each may request a priority
  - Multi-bound functions are executed from highest to lowest priority, and in an indeterminate order within the same priority
  - In the case of outputs, the last output to set the value wins
    - > There should probably be a mechanism for writers to know about each other and properly deal with this
- Some functions may have multiple (downward) bindings, such as a limit switch object that listens to multiple inputs

### Accessing Objects in Status Reports

- Any readable IO value can also be included in a status report as a "watch" value 
- Any time the value changes it will be reported in the next status report
- Some high-priority objects such as limits and alarms will invoke a status report if their value has changed  
- We highly recommend using the filtered status report option if you have a lot of watches  

### Accessing Objects via Gode

Some IO commands and functions make sense to operate from within Gcode and thus be part of the job and synchronized with motion. We use JSON active comments, which is a Gcode comment that gets executed. In standard Gcode the string "MSG" occurring right after the open paren is an active comment that sends the comment to the console, e.g. `G0 X100 (MSG Hello World). JSON active comments rely on an open curly immediately after the open paren.

- `M100 ({...})` - JSON command synchronized with motion. The JSON command, presumably an output, will be executed when the planner gets to it.
- `M101 ({...})` - Wait on event. The planner will hold until the JSON command, presumably an input, returns `true`.
