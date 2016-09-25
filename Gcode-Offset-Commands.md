
##G10 L2 Set Parameters (Offsets)
The G10 L2 command is used to set coordinate offsets. Use Pn to select coordinate system 1-6 (G54 - G59, respectively), and one or more axis values to set the offset for that axis. The axis value is the offset you want for that axis, relative the zero set during homing. For example, if you wanted the X origin of coordinate system 1 (G54) to be 100 mm to the right of zero you would enter G10 L2 X100.

##G10 L20 Set Parameters (Offsets)
The G10 L20 command is also used to set coordinate offsets. Use Pn to select coordinate system 1-6 (G54 - G59, respectively), and one or more axis values to set the offset for that axis. The axis value is the value you wish the current point to be. For example, if you wanted the current X location to be X0 in coordinate system 1 (G54) you would enter G10 L20 X0.