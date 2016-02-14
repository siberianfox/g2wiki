**This page discusses coordinate systems which can be source of endless confusion**

##Coordinate Systems and Offsets
TinyG supports these coordinate, offsets and positioning commands that are all related:

* G10 L2 - Set coordinate offsets 
* G53 - Absolute coordinate system (aka machine coordinate system)
* G54 - Coordinate system 1 - set offsets using G10 L2 P1 X...
* G55 - Coordinate system 2 - set offsets using G10 L2 P2 X...
* G56 - Coordinate system 3 - set offsets using G10 L2 P3 X...
* G57 - Coordinate system 4 - set offsets using G10 L2 P4 X...
* G58 - Coordinate system 5 - set offsets using G10 L2 P5 X...
* G59 - Coordinate system 6 - set offsets using G10 L2 P6 X...
* G92 - Temporary offsets (G92.1, G92.2, G92.3, G92.4)
* G28.1 - Saves the absolute position and G28 returns to the saved position
* G30.1 - Saves the absolute position and G30 returns to the saved position
* G28.2 - Set absolute coordinates using homing cycle
* G28.3 - Set absolute coordinates using axis words in the gcode command
* G38.2 - Probe for a position

###Absolute Coordinate System (G53)
The coordinate systems and their offsets can be viewed in layers. The bottom layer is the **absolute** coordinate system aka **machine** coordinate system. This is always the G53 coordinate system, and can be selected by including G53 on the Gcode line. G53 is non-modal, meaning that it only applies to the line where it is present.

The absolute coordinate system can be set a few different ways. The G28.2 homing operation runs a homing cycle for the axes specified, at the end of which the absolute coordinate system will be (or is supposed to be) referenced to the homing switches. In most machines the X/Y zero will be at the front left hand corner of the machine. Preferences for Z zero vary, but they are typically at either the top or bottom of travel. G28.2 is not an official Gcode - it's just something we implemented in TinyG.

G28.3 also sets absolute coordinates, but without running a homing cycle. It sets the axis or axes specified to the axis words provided. This is useful for infinite axes and other special cases. For example, a particular cardboard cutter has a pair of pinch rollers to feed cardboard in and out on the Y axis. Asking the user to insert the cardboard then press a button can send G28.3 Y0 to set the Y axis to zero at the edge of the cardboard. G28.3 is another TinyG-only code.

G28 and G30 also use absolute coordinates, as explained later.

It's good practice to set the absolute coordinate system then leave it alone. I.e. don't move it around just to center the work piece. Use offsets for that. See below...

###Coordinate Systems and Offsets (G54-G59)
At all times other than when a G53 is active the machine will be in one of the 6 coordinate systems (1-6), corresponding to G54-G59. These allow you to set offsets on any or all axes using the G10 L2 command. If the offsets are zero the coord system will be the same as the G53 system (i.e. in absolute coordinates), but it's persistent, so you don't need to put the coordinate system on every Gcode line. It's common practice to leave the G54 offsets at zero so you have a persistent machine coordinate system available.

Another common practice is to set one of the coordinate systems to the middle of the work area, let's say G55. If the table has dimensions of 400mm x 400mm (X/Y) then issuing `G10 L2 P2 X200 Y200` will set coordinate system 2 (G55) to the middle of the X/Y table. When a move G0 X0 Y0 is issued the offsets are added to the absolute coordinates, so the move will go to the middle of the X/Y table.

Another use of coordinate offsets is to locate the tool position in Z. If you do a G38.2 probe (see later) the resulting position can be used to set an offset in Z that locates the probe position at 0, all without messing with the absolute coordinate system.

G10 offsets are persistent, that is they are saved to EEPROM once the machining cycle is done. So you can set them up once and use them for multiple jobs.

##Offsets to the Offsets (G92)
G92 provides **temporary** offsets that can be applied on top of the coordinate system offsets. So now we are 3 layers deep. G92 sets temporary offsets similarly to the way G28.3 works. It sets the current position to some value you provide. If you want the current position to be seen as 0,0,0, then issue `G92 X0 Y0 Z0`. But instead of changing absolute coordinates it just computes the offsets on to of the currently active coordinate system. G92.1 un-does the offsets and returns you to the value in the current coordinate system. If you change the coordinate system the offsets remain the same and the position you set in the G92 will shift by the difference between the 2 systems (confused yet? Try programming this.)

G92 offsets are temporary. They are not saved to EEPROM, and they are forgotten when you do a cycle end (M2 or M30)

###G28/G30 Go To Position
Now that the offsets are clear as mud, let's discuss G28 and G30. These are actually quite simple. G30 is just another G28, so for this discussion we'll just talk about G28. G30 works identically.

To 'save' a position just send `G28.1`. This saves the current position in absolute coordinates - regardless of the coordinate system the machine is in and the state of the G92 offsets. When you want to return to that position send `G28`. 

If you send axis words along with the G28, say `G28 Z10` the tool will move to the intermediate position first, then move to the saved position. This is useful for clearing the work before moving. G28 is a good choice for tool changes and things like that.
