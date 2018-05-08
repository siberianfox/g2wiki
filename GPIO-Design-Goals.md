This page discusses upcoming functionality being added to the GPIO system.<br>

### Pages:
- [GPIO Overview](GPIO-Design-Discussion)
- GPIO Design Goals
- [GPIO Primitives](gpio-primitives)
- [GPIO Objects and Bindings](gpio-objects-and-binding)
- [GPIO Implementation](gpio-implementation)

See also:
- [G2 GPIO System](https://github.com/synthetos/g2/wiki/Digital-IO-(GPIO))
- [JSON Operation](https://github.com/synthetos/g2/wiki/JSON-Operation)
- [Toolheads](https://github.com/synthetos/g2_private/wiki/Toolheads)

# Design Goals
In short, we want the GPIO system to be general enough to connect any pin to any machine function as a configuration option. Practically speaking, some pins like motor drive signals will be hard wired, but everything else should be configurable.

We also want different physical machine and board configurations to be more portable and consistent across different UIs, and be able to be driven from the same Gcode and JSON regardless of differences in the underlying hardware or configuration. 
- For example, we could have several machines where `in0` is always valid and serves the same function (like a shutdown signal), but on one machine it's attached to `di0` but on another it's attached to `di5`. 
- Better yet, create a `shutdown` object and listen to whatever pin or pins invoke that object, regardless of the machine configuration. Another example would be where "heater PWM" for Tool 1 (extruder 1) is using `do0` whereas Tool 2 uses `do7`. We want to hide these details from the UI.
- We also want to be able to create and control arbitrary "tools" like extruders or vacuum nozzles and still be as true to classic Gcode as possible. The GPIO system should provide the ability to drive arbitrary devices using standard Gcode commands.

### Design Goals In Brief
- Provide a flexible way to expose, configure and control low-level digital and analog IO
- Provide logical-to-physical mapping of IO ports to the objects that use them
- Provide a way to assign (bind) arbitrary functions to arbitrary IOs
- Support concept of "signals" and virtual IO devices
  - Allow g2 objects to generate and consume digital and analog values that can be wired to physical pins and IO devices
- Support concept of "events"
  - ...which are changes of state of signals
- Make assignment and bindings of pins and signals configurable at runtime (as much as practical). I.e. minimize the number of hard-wired or compile-time choices in favor of making these choices configurable through the interface
- Support the concept of generalized Tools and Toolheads with assignable functions
- Provide a framework for tools and other devices made of composable objects

#### Footnote: Nomencalture
- **Object**: We use 'object' loosely. An object is any g2 code that performs some function, like raise an alarm if a limit is hit, or detect if a safety door is open. An object may be complex and expose multiple member functions, like a heater object that may expose functions to detect that a heater has reached a stable set temperature, and the state of a PID loop. 
- **Host**: A host is some processor above the motion controller, such as a single board Linux machine or a desktop computer.

### Must Support these Use Cases
- **Read Physical Input Pins**
  - Host can read IO pins directly 
    - Ex: Read an on/off signal from an external device connected to a pin
  - Host can display and/or take action on these state changes
- **Write Physical Output Pins**
  - Host can set IO pins directly. 
    - Ex: Turn on/off a pin to actuate an annunciator connected to a pin
  - Host can monitor these output states to display and/or take action on state changes
- **Read Objects**
  - Host can monitor internal g2 objects (i.e. can read output signals from code)
    - Ex: Monitor limit and alarm conditions
    - Ex: Detect spindle at-speed or temperature at set-point
  - Host can display and/or take action on object state changes
- **Write Objects**
  - Host can control internal g2 objects (i.e. can send signals to code)
    - Ex: Turn on/off coolant - where the object is the coolant control
    - Ex: Turn on/off, control direction and speed of spindle
    - Ex: Control heaters, fans and other devices
  - Host can read back the signal(s) it has sent to an object
- **Wire Inputs and Outputs to Internal Objects**
  - An object can use a physical or logical IO as an input
    - Ex: Use input pin to trigger limit or alarm - e.g. pin3 is Y-min limit switch
    - Ex: Use signal output by another object as an input - e.g. detect heater at set temperature
  - An object can produce an output that is "wired" to a pin. 
    - Ex: A watchdog timer is wired to a SAFE pin that asserts if the timer ever expires
- **Multiple IOs to an Object**
  - Multiple primitives can feed an object
    - Ex: A variety of input switches will trigger an alarm or a limit object
- **Multiple Objects using an IO**
  - Multiple objects can read or set a single IO
    - One object changes the input, and multiple objects can read and act on this change
    - One output "signal" can be applied to multiple pins
  - Logically this looks like a very simple publish and subscribe system
  - Internally these are chained in a list - so a priority is inherent
- **IO Addressable through Tools and Toolheads**
  - Support a machine with a mix of tool types, any or all of:
    - Spindle that can accommodate multiple end mills and probes
    - Extruder that can have more than 1 color
    - Changeable extruders (like an ATC for extruders)

### Some Example Machine Configurations to Support
- Classic milling machine with a spindle and manual tool changes
- Classic milling machine with a spindle and automatic tool changer (ATC)
- Classic filament 3d printer with a single extruder
- Filament 3d printer with dual extruders on the same gantry
- 3D filament printer with a manual procedure for changing filament mid-job
- 3D filament printer with multiple, changeable extruders
- Laser cutter with air cooling
- Pick and Place machine with a fixed vacuum pick nozzle
- Pick and Place machine with changeable vacuum pick nozzles
