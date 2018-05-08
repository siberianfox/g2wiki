This page discusses toolhead concepts and implementation.<br>

See also:
- [G2 GPIO System](Digital-IO)
- [JSON Operation](JSON-Operation)

# Toolheads and Tools
In GCode a tool is "the thing that's moved around by the machine." However, since a tool needs something to hold it, we refer to the part that moves around and holds the tool as a "toolhead." 

A tool is defined by a tool number 1-32, which also specifies the entry position in the tool table. The tool table has one entry for each tool, and specifies the toolhead and the related toolhead parameters for this specific tool. The  parameters of the tool table entry may vary widely for a given type of tool (e.g. spindle vs. extruder vs. vacuum pick head). 

Some tools are changeable during the job, some are changeable as part of job setup, some are fixed and cannot be changed. Some examples:
- **Changeable during the job**: A spindle with indexed end-mills and an automatic tool changer (ATC) or a manual tool change routine
- **Changeable during job setup**: A 1.75mm extruder with a specific filament. The filament and its parameters are part of the tool table. If a different filament were used it would be a different tool with different parameters. The filament cannot be changed once the job has been started.
- **Not changeable**: A laser cutter with a dye laser. The parameters describing the laser are in the tool table, but they are not expected to be changed.
- Example toolheads (for the following example tool table):
  - Toolhead 1:
    - Type: Spindle
    - Spindle speed control: (PWM) output on DO3
    - Coolant output control: DO4
    - Tool Offset: {0, 0, 0}
  - Toolhead 2:
    - Type: Extruder
    - Heater: HE2 (configured separately)
    - Coolant (fan) output control: DO5
    - Tool Offset: {0, -20, -5}
    - Max filament diameter: 1.85 (mm)
- Example tool table:
  - Tool 1:
    - Toolhead: 1
    - Name: "1/2in endmill"
    - Tool offset: {0, 0, -5} (Z length offset)
    - Diameter: 12.7 (mm)
  - Tool 2:
    - Toolhead: 1
    - Name: "1/4in endmill"
    - Tool length offset: {0, 0, -4}
    - Diameter: 6.35 (mm)
  - Tool 3:
    - Toolhead: 2
    - Name: "Colorfabb Blue"
    - Filament Diameter: 1.74

## Toolheads and Gcode
All toolheads have a type, such as spindle, extruder, etc., or something more fine-grained, like a serial controlled spindle. The toolhead type selected specifies the semantics of certain Gcodes for that class of tools. The way the Gcode is interpreted by the toolhead is the contract between Gcode and the toolhead and should not be violated.

Specific Gcodes that may be implemented and/or affected include:
  - `S nn.nn` is "speed". This is the primary parameter for the tool
    - Spindles interpret `S` as "RPM"
    - Extruders interpret `S` as milliliters deposited per mm of linear travel
    - Lasers interpret `S` as output power.
  - `S0` is the zero semantic. For example, in a spindle, `S0` should act as `M5`
  - `G0` Tool behavior during traverse - e.g. turn laser off during traverses
  - `G40` Cutter compensation off
  - `G41` Cutter compensation left & `D` parameter. Also `G41.1`
  - `G42` Cutter compensation right & `D` parameter. Also `G42.1`
  - `G43` Tool length offset. Also `G43.1`, `G43.2`
  - `G49` Cancel tool length compensation
  - `G96` Constant Surface Speed and `D` and `S` parameters; or alternate interpretation of `S` word
  - `G97` RPM Mode or the "normal" interpretation of `S` word
  - `M3` activates the tool with "direction" parameter = 1
    - Spindles interpret M3 as start clockwise rotation. A delay-before-motion may be implicit and optionally added.
    - Extruders interpret M3 as start extruding proportionally with XYZ motion. A priming motion ("un-retraction") may be implicit and optionally added if needed.
    - Lasers interpret M3 as "laser on."
  - `M4` activates the tool with "direction" parameter = 0
  - `M5` deactivates the tool. May invoke spin-down or other toolhead-specific behaviors
  - `Tn` selects tool but does not change to it. May invoke tool-specific behaviors such as pre-heat
  - `M6` initiates a tool change - which may be a sequence specific to the toolhead type
  - `M7` activates coolant 1 (by convention mist coolant). Invoke tool-specific actions such as vacuum cooling 
  - `M8` activates coolant 2 (by convention flood coolant)
  - `M9` deactivate all coolants
  - `M19` orient spindle & `R`, `Q`, `P` parameters

### Toolheads, Gcode and JSON Active Comments
Any of the above Gcodes can be overloaded w/active comments w/parameters as needed for a given tool type. As an example, for a given extruder toolhead `M7` (mist coolant, coolant 1) controls the work fan (down fan). The program may want to modulate the fan speed, but Gcode M7 does not accept a parameter. The Gcode command could be extended with a JSON active comment e.g. `M7 ({s:0.75})` to set fan speed to 75%. The namespace for the M7 command is used to confine its valid Gcodes to `s`, which in this example is the only command it knows how to handle. The namespacing also keeps M7's use of `s` from colliding with any other use of `s`.

### Direct and Implicit Addressing
Objects may be directly addressed as JSON keys in the normal way, as described [here](JSON-Operation). Bindings and encapsulation in tools and toolheads also supports implicit addressing that can generalize and simplify the calling JSON or Gcode. 

For example: A 3D printer system has two heaters, `he1` is the extruder heater and `he2` is the heated bed. These are described by objects that can be accessed directly as `{he1...:n}` and `{he2...:n}`. In addition, since the extruder's heater component is encapsulated by the tool component it can be addressed implicitly, or simply as `he` in that context: `{t3:{he:n}}`. If T3 is the currently active tool (from an M6) then this can be reduced to `{t:{he:n}}`. This notation allows commands to be written generally, without being required to know the direct device-level addressing, or without having to know the tool number of the currently active tool.

Notes:
- Alternately we could do this by reserving zero for the implicit address. In which case the above examples become `{t3:{he0:n}}` and `{t0:{he0:n}}`. This is more lexically complex but has the advantage that we can more easily deal with multiple similar devices (see below).
- We need a way to handle implicit addressing for multiple similar devices. For example 2 or more temperature sensors in an extruder, call them `{te1`, `{te2`, `{te3`. We may want `{te:n` to be the temperature sensor group, in which case the `{te:n` method for implicit addressing won't work.
- In implementation we may want to simply have a key:value pair that defines the mapping of the implicit address to the direct address. In this case the rules we use are flexible. But we still need some conventions to manage implicit naming.
