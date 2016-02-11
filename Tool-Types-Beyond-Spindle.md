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

We change everything! (TBD)