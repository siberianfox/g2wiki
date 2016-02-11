# Purpose

We would like to consider expanding the concept of a toolhead in ways that are mostly compatible with NIST-style gcode without a pletohora of additional M-codes or the need to unnecessarily re-interpret the meaning of axes.

As a secondary goal, we would like to encourage and support the use of "device-independent gcode." This primarily means that the gcode designed for one type of machine (say an FFF 3D printer) should work on any other machine of the same type and with the same used set of features. There will of course be exceptions (a gcode file that uses 2 extruders won't work on a machine with one, for example), but we can still eliminate the incorporating of _transient settings_ into the gcode, such as filament diameter.

