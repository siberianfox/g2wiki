This page discusses the GPIO system in the upcoming 1.0.0 release<br>

### Pages:
- GPIO Overview
- [GPIO Design Goals](gpio-design-goals)
- [GPIO Primitives](gpio-primitives)
- [GPIO Objects and Bindings](gpio-objects-and-binding)
- [GPIO Implementation](gpio-implementation)

See also:
- [G2 GPIO System](Digital-IO)
- [JSON Operation](JSON-Operation)
- [Toolheads](Toolheads)

# Overview
## Overview of System Model
General Purpose IO is the basis of the g2 system model. 

**IO Primitive** is the lowest layer, consisting of physical input and output pins. Primitives also include inputs and outputs from internal g2 objects - i.e. code that generates and consumes **signals** and **events** (where an event is a change in the state of a signal).

**Signal** is any value that is communicated. It may be binary (0,1), or analog ranging 0.000 - 1.000. An input switch generates a binary signal. An analog-to-digital converter generates an analog signal. An object that waits until a heater is within regulated temperature may generate a signal as a binary output.

**Event** is a transition of a signal, such as an input going from inactive to its active state, or an analog value crossing some threshold.

**Exposed-as** is how a primitive IO can be exposed. primitive IOs can be `inM`, `outM`, and `ainM` (analog input).

**Objects** We use the term 'object' loosely. Objects are "_any code in g2 that consumes or generates a signal that we can connect to_". Objects read (input) and write (output) signals. Exactly what the object is or does is less important than the fact that we have a configurable interface to it. Objects may do something simple and expose only a single function; like react to a limit switch being hit, or trigger an alarm state. An object can be complex and expose multiple member functions; like a PID loop that consumes an analog temperature signal and produces a PWM power signal.

**Bindings** are how you connect to objects; for example a limit object can be bound to one of more inputs (IO primitives) that are wired to limit switches.

A **Component** is a collection of one or more objects that does something. In some cases the component is very simple and only has one function. Like a component to read an input and report it out as, say, a door-is-open signal. In other cases a component may be more complex, encapsulating multiple objects that perform multiple functions. A heater is an example of a complex component, as it has PWM outputs, temperature inputs, PID processing, timeout functions, output signals and a variety of settings. Components are composable - they may be part of other components. A heater component may be part of an extruder component.

Finally, **Toolheads** are specialized components.  A [toolhead](Toolheads) is the thing that is moved around by the machine - like a spindle or an extruder. Toolheads can hold different **Tools**. This is an important concept for generalization and is described in more detail later.

## System Model - More Details
We want a simple but flexible way to connect system objects to actual physical IO. We also want to be as true to the use of Gcode as possible. So we leverage the Gcode Tool concept to support anything that can be moved around in space. With these goals in mind the GPIO system is layered - from lowest layer to highest:

- **[IO Primitive](#io-primitives)** is the actual hardware or an input/output of a g2 object
  - An IO Primitive refers to the physical layer and consists of digital inputs, analog inputs, and digital outputs
  - IO primitives can also be virtual signals, such as a timer expiring or a communications operation with a remote device; e.g. a command to turn on a temperature controller sent over a serial bus.
  - Primitives are configured using their associated configurators
    - _`diN`_ (configure digital input N)
    - _`aiN`_ (configure analog input N)
    - _`doN`_ (configure digital output N)
  - Primitives may be _Exposed As_ simple read and write objects: 
    - _`inM`_  (read digital input M)
    - _`ainM`_ (read analog input M - i.e. an ADC function)
    - _`outM`_ (read and write digital output M - some outs can also do PWM; effectively an analog output as well)
  - Note that the mapping of the physical input `N` to the logical output `M` is configurable

- **[Objects](#objects-and-binding)** refers to the logical layer
  - The GPIO system provides a scheme for flexibly binding IO to objects, providing a logical-to-physical mapping
  - _Objects_ are things like trip-limit, start-feedhold, spindle-on-CCW, set-extrusion-rate, etc.
  - Objects may be simple, like trip-limit, start-feedhold, or inM exposed-as object
  - ...or they may be complex things with multiple member functions such as an extruder object exposing functions for set-temperature, at-temperature (a threshold function), get-fan-power, etc.

- **[Toolhead](#toolheads-and-tools)** provides a way to collect objects together to act as a tool component; like a spindle or an extruder or a cutting laser.
  - A toolhead is something that is moved around in XYZ at the "controlled point"
  - A spindle is the classic toolhead for CNC, carrying an end mill and using `S` as its speed parameter
  - Tool heads set the semantics of how Gcode is interpreted to be meaningful for a given tool, and allows us to to adhere to the standard semantics of Gcode for a variety of tool types. The Gcode interpretation becomes modal for the active tool.
  - Each toolhead type exposes different parameters that have a reasonable default, but a tool may override.
  - Some examples of toolheads are: spindle, extruder, laser, probe, heated wire, vacuum head (pick and place), scanning camera, paste dispenser, RF probe, etc.
  - Importantly, the toolhead construct allows us to build a multi-modal machine with different types of toolheads and drive a job from a single Gcode file. The modality of the machine changes as toolheads are activated (M6).

- **[Tool](#toolheads-and-tools)** is something that is carried by a toolhead.
  - Different types of tools are used by different types of toolheads - e.g. an end mill for a spindle, a type of filament for an extruder.
  - Selecting a tool also selects the toolhead that the tool is bound to in the tool table.  
  - The tool table table provides a way to define the parameters of the tool, and to choose a toolhead and set any of the parameters that should be used for that tool/toolhead combination
  - Some examples of tools are: end mill (spindle), drill bit (spindle), filament (extruder)
  - Tools carry a number from 1 to 32. The system supports accessing and configuring the tool table in JSON, and changing to a numbered tool during a job.
