
`G2 or G3 'axes' 'offsets' Pn Fnnn` center format arc<br>
`G2 or G3 'offsets' Pn Fnnn` center format full circle arc<br>
`G2 or G3 'axes' Rnnn Pn Fnnn` radius format arc<br>
`G2 or G3 Rnnn Pn Fnnn` radius format full circle arc<br>

- 'axes' are any of XYXABC
- 'offsets' are any of IJK

_Note: Much of this specification has been copied from [Linux CNC Gcode, G2-G3](http://linuxcnc.org/docs/html/gcode/g-code.html#gcode:g2-g3), but they are not identical as some differences exist._

A circular or helical arc is specified using either G2 (clockwise arc) or G3 (counterclockwise arc) at the current feed rate `F`. The direction (CW, CCW) is as viewed from the positive end of the axis about which the circular motion occurs.

- The axis of the circle or helix must be parallel to the X, Y, or Z axis of the machine coordinate system. The axis (or, equivalently, the plane perpendicular to the axis) is selected with G17 (Z-axis, XY-plane), G18 (Y-axis, XZ-plane), or G19 (X-axis, YZ-plane). If the arc is circular, it lies in a plane parallel to the selected plane.

- To produce a helix include the axis word perpendicular to the arc plane: for example, if in the G17 plane, include a Z word. This will cause the Z axis to move to the programmed value during the circular XY motion.

- To program an arc that gives more than one full turn use the P word specifying the number of full turns plus the programmed arc. The P word must be an integer. If P is unspecified, the behavior is as if P1 was given: that is, only one full or partial turn will result. For example, if a 180 degree arc is programmed with a P2, the resulting motion will be 1 1/2 rotations. For each P increment above 1 an extra full circle is added to the programmed arc. Multi-turn helical arcs are supported and give motion useful for milling holes or threads.

- If a line of code makes an arc and includes rotary axis motion the rotary axes turn at a constant rate so that the rotary motion starts and finishes when the XYZ motion starts and finishes. Lines of this sort are hardly ever programmed.

- The arc center is absolute or relative as set by G90.1 or G91.1 respectively.

- Two formats are allowed for specifying an arc: Center Format and Radius Format.

It is an error if:

- No feed rate has been set
- The P word is not an integer

###Center Format Arcs
Center format arcs are more accurate than radius format arcs and are the preferred format to use.

- The end point of the arc ('axes') along with the offset to the center of the arc from the current location ('offsets') are used to program arcs that are less than a full circle

- It is OK if the end point of the arc is the same as the current location, but these arcs will have no movement. They are not treated as a full circle arc. For full circle arcs the endpoints must be omitted.

- The offset to the center of the arc from the current location and optionally the number of turns are used to program full circles. For full circle arcs no 'axes' words should be provided.

- When programming arcs an error due to rounding can result from using a precision of less than 4 decimal places (0.0000) for inch and less than 3 decimal places (0.000) for millimeters.

####Incremental Arc Distance Mode
Incremental Arc Distance Mode is the default mode, and may be set by `G91.1`. The following applies:

- Arc center offsets are a relative distance from the start location of the arc
- One or more axis words and one or more offsets must be programmed for an arc that is less than 360 degrees
- No axis words and one or more offsets must be programmed for full circles. The P word defaults to 1 and is optional

####Absolute Arc Distance Mode
Absolute Arc Distance Mode may be set by `G90.1`. The following applies:

- Arc center offsets are the absolute distance from the current 0 position of the axis
- One or more axis words and both offsets must be programmed for arcs less than 360 degrees
- No axis words and both offsets must be programmed for full circles. The P word defaults to 1 and is optional

####XY-Plane Arcs (G17)
`G2 or G3 Xnnn Ynnn Znnn Innn Jnnn Pn`

  - X, Y - Endpoint, or omitted for full-circle arc
  - Z - helix travel (optional)
  - I - X offset
  - J - Y offset
  - K - Z offset MUST NOT BE PRESENT
  - P - number of turns (optional, must be integer)

####XZ-Plane Arcs (G18)
`G2 or G3 Xnnn Znnn Ynnn Innn Knnn Pn`

  - X, Z - Endpoint, or omitted for full-circle arc
  - Y - helix travel (optional)
  - I - X offset
  - J - Y offset MUST NOT BE PRESENT
  - K - Z offset
  - P - number of turns (optional, must be integer)

####YZ-Plane Arcs (G19)
`G2 or G3 Ynnn Znnn Xnnn Jnnn Knnn Pn`

  - Y, Z - Endpoint, or omitted for full-circle arc
  - X - helix travel (optional)
  - I - X offset MUST NOT BE PRESENT
  - J - Y offset
  - K - Z offset
  - P - number of turns (optional, must be integer)

For any of the the G17, G18, G19 arcs, it is an error if:

- No feed rate is set with the F word
- No offsets are programmed
- An illegal offset is present
- When the arc is projected on the selected plane, the distance from the current point to the center differs from the distance from the end point to the center by more than (.05 inch/.5 mm) OR ((.0005 inch/.005mm) AND .1% of radius).

###Radius Format Arcs
`G2 or G3 Xnnn Ynnn Znnn Rnnn Fnnn`<br>
`G2 or G3 Rnnn Pn Fnnn`<br>

In the radius format the coordinates of the end point of the arc in the selected plane are specified along with the radius of the arc. 

- The axis words are optional except at least one of the two words for the axes in the selected plane must be used 
- The R number is the radius
- A positive radius indicates that the arc turns through less than 180 degrees, while a negative radius indicates a turn of more than 180 degrees
- If the arc is helical, the value of the end point of the arc on the coordinate axis parallel to the axis of the helix is also specified.

It is an error if:

- Both of the axis words for the axes of the selected plane are omitted
- The end point of the arc is the same as the current point

It is not good practice to program radius format arcs that are nearly full circles or nearly semicircles because a small change in the location of the end point will produce a much larger change in the location of the center of the circle (and, hence, the middle of the arc). The magnification effect is large enough that rounding error in a number can produce out-of-tolerance cuts. For instance, a 1% displacement of the endpoint of a 180 degree arc produced a 7% displacement of the point 90 degrees along the arc. Nearly full circles are even worse. Other size arcs (in the range tiny to 165 degrees or 195 to 345 degrees) are OK.