# Purpose

We would like to consider expanding the concept of a toolhead in ways that are mostly compatible with NIST-style gcode without a pletohora of additional M-codes or the need to unnecessarily re-interpret the meaning of axes.

As a secondary goal, we would like to encourage and support the use of "device-independent gcode." This primarily means that the gcode designed for one type of machine (say an FFF 3D printer) should work on any other machine of the same type and with the same used set of features. There will of course be exceptions (a gcode file that uses 2 extruders won't work on a machine with one, for example), but we can still eliminate the incorporating of _transient settings_ into the gcode, such as filament diameter.

# Background

In current gcode (not counting extensions such as those used by RepRap flavors of 3d Printer, etc), there is the concept of an ever-present "spindle". That spindle may also make use of a "tool changer" to change the bit held by the tool (or possibly the entire spindle itself). These tools start at index 1 and go up, with 0 being a special meaning "empty spindle" or "no tool".

Usage example:

```gcode
G0 X0 Y0      ; move to (0,0) quickly
M3 S12000     ; Turn on the spindle, rotating clockwise at "12000"
...           ; moves while the spindle is on
G1 ... S11000 ; move to ... after adjusting the speed to "11000"
M5            ; turn spindle off
```

There are other traditional CNC facilities, such as "coolant" that may be reappropriated as well.

# Proposal

We change the term "spindle" to "toolhead" or just "tool." Then we say that there are different types of tools, one of which is a spindle. 

Types of toolhead:
- **Spindle**
  - *`M3`, `M4`, `M5` behavior:* Turn on the spindle at the RPM specified by the S word.
    - Motion will plan to a stop for `M3`, `M4`, and possibly S word changes, with a configurable delay to allow the RPM to be achieved.
    - Some spindles will have a signal to indicate that they are at speed, in which case we could wait for that signal.
- **Laser**
  - *`M3`, `M4`, `M5` behavior:* Turn on a laser, with the power setting based on the S word, possibly with some coversion table between input and output.
    - Motion will not have to be paused for laser enabling or power level changes.
    - Power level applied is to be adjusted by the active velocity dynamically (on a segment-by-segment basis) to maintain consistent laser coverage even during acceleration and deceleration.
- **Extruder**
  - *`M3`, `M4`, `M5` behavior:* The `S` word is interpreted as mm³ (or microliters, uL). All feed moves (G1, arcs, etc) will move the associated axis motor by the appropriate amount in order to extrude that uL of filament per linear mm of travel.
    - An otherwise configured or measured filament diameter will be used in the computation of how far to move that extruder axis.
    - The extruder axis should be one of U, V, or W linear axis. The axis should not be used directly *except* for the case of loading filament.
    - Filament retraction can be handled internally and driven by `M3`, `M5`, or the use of one or more `G0`. Retraction should not happen without a minimal configurable travel.


# Tool switching

Each tool number can have a different type. This allows multiple of one type (2 extruders) or different types on a machine (extruder and spindle).

Tool switching will be controlled by `M6` and a `T` word.
