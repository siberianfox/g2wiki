_Applies to firmware build 100.xx and above_

_[<<< Back to Gcode Main Page](Gcodes)_

##G10 Ln Set Parameters

###G10 L2 Set Coordinate System Offsets - Relative to Machine Origin
`G10 L2 Pn 'axes'` set coordinate system offsets relative to machine origin

G10 L2 offsets the origin of the axes in the specified coordinate system to the value of the axis word.

- P is coordinate system 1-6, corresponding to G54 - G59, respectively
- The offset values in the 'axes' words are from the machine origin established during homing
- Axis words not used will not be changed
- The offset value will replace any current offsets in effect for the specified coordinate system
- G10 L2 Pn does not change from the current coordinate system to the one specified by P, you have to use G54-59
- G10 L2 Pn offsets set are independent of G92 origin offsets, and are cumulative
- The coordinate system set by a G10 command may be active or inactive at the time the G10 is executed. If it is currently active the new coordinates take effect immediately

For example, if you wanted the X origin of coordinate system 1 (G54) to be 100 mm to the right of zero you would enter G10 L2 X100.

###G10 L20 Set Coordinate System Offsets - Relative to Current Position
`G10 L20 Pn 'axes'` set coordinate system offsets relative to the current position

The G10 L20 command is also used to set coordinate offsets. Use Pn to select coordinate system 1-6 (G54 - G59, respectively), and one or more axis values to set the offset for that axis. The axis value is the value you wish the current point to be. For example, if you wanted the current X location to be X0 in coordinate system 1 (G54) you would enter G10 L20 X0.