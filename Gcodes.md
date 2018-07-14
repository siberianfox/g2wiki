**NOTE: This page describes Gcode supported by g2core in firmware builds 100.xx and later**<br>

G2 implements the NIST RS274v3/ngc dialect of Gcode informed by the LinuxCNC Gcode specification:
- [Kramer's NIST RS274NGCv3 Gcode Specification](http://technisoftdirect.com/catalog/download/RS274NGC_3.pdf)
- [LinuxCNC Gcode Specification](http://www.linuxcnc.org/docs/2.4/html/gcode_main.html)<br>

This page and related pages describe basic Gcode commands for movement and parameters. Much of this is shamelessly cribbed from the [LinuxCNC Gcode pages](http://linuxcnc.org/docs/html/gcode/g-code.html)<br>

Related Pages
- [Mcodes](Mcodes)
- [G2, G3 Arc At Feed Rate](G2-G3-Arc-At-Feed-Rate)
- [G38.x Probes](Gcode-Probes)
- [Coordinate Offsets](Gcode-Coordinate-Offsets) and [Coordinate Systems](Cooordinate-Systems) for G10, G53-G59, G92.x, JSON Offset commands

## Gcode Summary
This table summarizes Gcode supported. In the following _'axes'_ means one or more of X,Y,Z,A,B,C, along with a corresponding floating point value for a specified axis.

Gcode | Parameters | Command | Description
------|------------|---------|-------------
__________|___________|_____________________|
[G0](#g0-straight-traverse-rapid-move) | _axes_ | Straight traverse | Traverse at maximum velocity
[G1](#g1-straight-feed-cutting-move) | _axes_ F | Straight feed | Move at feed rate F
[G2](G2-G3-Arc-At-Feed-Rate) | _axes_ F P  IJK or R | Clockwise arc feed | Arc at feed rate F
[G3](G2-G3-Arc-At-Feed-Rate) | _axes_ F P  IJK or R | Counterclockwise arc feed | Arc at feed rate F
[G4](#g4-dwell) | P | Dwell | Pause for P seconds
[G5](#g5-reserved-for-spline-motion) | | | Reserved for Spline Motion
[G10 L2](Gcode-Coordinate-Offsets#g10-ln-set-parameters) [G10 L20](Gcode-Coordinate-Offsets#g10-ln-set-parameters)  | _axes_, P | Set coord offsets
[G17](Gcode-Circular-Arcs#g17-g18-g19-select-arc-plane) | | Select XY arc plane |
[G18](Gcode-Circular-Arcs#g17-g18-g19-select-arc-plane) | | Select XZ arc plane |
[G19](Gcode-Circular-Arcs#g17-g18-g19-select-arc-plane) | | Select YZ arc plane |
[G20](#g20-g21-select-units-mode) | | Select inches mode | All Gcode from this point on will be interpreted in inches
[G21](#g20-g21-select-units-mode) | | Select mm mode | All Gcode from this point on will be interpreted in millimeters
[G28](#g28-g30-go-to-predefined-position) | _axes_ | Goto G28 position | Optional axes specify an intermediate point
[G28.1](#g28-g30-go-to-predefined-position) | | Set G28 position | The current machine position is recorded (No parameters are provided)
[G28.2](https://github.com/synthetos/TinyG/wiki/Homing-and-Limits-Description-and-Operation#g282---homing-sequence-homing-cycle) | _axes_ | Homing Sequence | Homes all axes present in command. At least one axis must be specified
[G28.3](https://github.com/synthetos/TinyG/wiki/Homing-and-Limits-Description-and-Operation#g283---set-absolute-position) | _axes_ | Set Absolute Position | Set axis to zero or other value. Use to zero axes that cannot otherwise be homed
[G30](#g28-g30-go-to-predefined-position) | _axes_ | Goto G30 position | Optional axes specify an intermediate point
[G30.1](#g28-g30-go-to-predefined-position) | | Set G30 position | The current machine position is recorded (No parameters are provided)
[G38.2](Gcode-Probes#g38x-probe) | _axes_ Fnnn | Probe | Probe toward, alarm if error
[G38.3](Gcode-Probes#g38x-probe) | _axes_ Fnnn | Probe | Probe toward, do not alarm if error
[G38.4](Gcode-Probes#g38x-probe) | _axes_ Fnnn | Probe | Probe away, alarm if error
[G38.5](Gcode-Probes#g38x-probe) | _axes_ Fnnn | Probe | Probe away, do not alarm if error  
[G53](Coordinate-Systems) | | Select absolute coordinates | Non-Modal: Applies only to current block
[G54](Coordinate-Systems) | | Select coord system 1 | G54 is typically used as the "normal" coordinate system and reflects the machine position
[G55](Coordinate-Systems) | | Select coord system 2 |
[G56](Coordinate-Systems) | | Select coord system 3 |
[G57](Coordinate-Systems) | | Select coord system 4 |
[G58](Coordinate-Systems) | | Select coord system 5 |
[G59](Coordinate-Systems) | | Select coord system 6 |
[G61](#g61-g64-path-control-modes) | | Set exact path mode | Continuous motion between Gcode blocks - exact path will be traced
[G61.1](#g61-g64-path-control-modes) | | Set exact stop mode | Motion will stop between each Gcode block
[G64](#g61-g64-path-control-modes) | | Continuous path mode | Same as exact path mode
[G80](#g80-cancel-motion-mode) | | Cancel motion mode |
[G90](#g90-g91-set-distance-mode) | | Set absolute distance mode |
[G90.1](#g90-g91-set-distance-mode) | | Set absolute arc distance mode |
[G91](#g90-g91-set-distance-mode) | | Set incremental distance mode |
[G91.1](#g90-g91-set-distance-mode) | | Set incremental arc distance mode | default arc mode
[G92](Coordinate-Systems) | _axes_ | Set origin offsets |
[G92.1](Coordinate-Systems) | | Reset origin offsets |
[G92.2](Coordinate-Systems) | | Suspend origin offsets |
[G92.3](Coordinate-Systems) | | Resume origin offsets |
[G93](#g93-g94-g95-feed-rate-mode) | | Set inverse feedrate mode |
[G94](#g93-g94-g95-feed-rate-mode) | | Set units-per-minutes feedrate mode / Cancel inverse feedrate mode |
[G95](#g93-g94-g95-feed-rate-mode) | | Reserved for "Set units-per-revolution feedrate mode" (unimplemented) |

 Other | Parameter |Command | Description
------|-----------|--------|-------------
N | line number | label gcode block | Line numbers are allowed, handled, and may be reported back in status reports. Don't underestimate how useful this is for debugging Gcode files.
[()](#gcode-comments) | [comment](#gcode-comments) | [gcode comment](#gcode-comments) | Gcode comments are supported. They are stripped and ignored, except for messages (below)
[;](#gcode-comments) | comment | alternate comment | A semicolon is an alternate way to delimit a comment. This is not Gcode "standard", but is used by Mach and some Reprap codes. (available as of build 378.05)
(msg....) | message | gcode message | Gcode messages are comments that begin with the characters `msg` (case insensitive). These will be echoed to the operator

## Gcode Comments
Gcode comments possess a number of features that extend their capabilities beyond classic Gcode. This is to support active comments and other developments. Please see [Gcode Comments](JSON-Active-Comments#gcode-comments)

## G0 Straight Traverse (Rapid Move)
`G0 'axes'`<br>
Straight traverse motion at maximum velocity

- All axis words are optional but must have a value if present. A traverse with no axes words will set motion mode to G0 but cause no movement
- `G0` is optional if the current motion mode is G0
- It is expected that cutting will not take place when a G0 command is executing

The velocity will be the vector sum of the axes participating in the move, limited such that no participating axis will exceed it's maximum velocity, `xVM` in the axis settings. In some cases resultant toolhead velocity may exceed the velocities of the individual axes. For example, a move from (0,0) to (100,100) where `xvm` = 10,000 mm/min and `yvm` = 10,000 mm/min will execute at 14,143 mm/min as the resulting movement vector is sqrt(2)/2 x 10,000 mm/min.

the achievable velocity may also be limited by the jerk settings `xJM` for each participating axis. This occurs in the case where the move cannot achieve its maximum velocity due to jerk limitations in one or more axes. Jerk is also computed as a vector sum so that none of the participating axes will exceed their maximum jerk settings during the move.

## G1 Straight Feed (Cutting Move)
`G1 'axes' Fnnn` causes straight feed motion at the requested feed rate or slower

- All axis words are optional but must have a value if present. A feed with no axes words will set motion mode to G1 but cause no movement
- `G1` is optional if the current motion mode is G1
- `F` is optional if a feed rate is in effect. It is an error if no feed rate is in effect and none is specified.

#### Calculation of Feed Rate for Moves Including Rotary Axes

G2 adheres to the [NIST Gcode definition](https://www.nist.gov/publications/nist-rs274ngc-interpreter-version-3) for calculating feedrate for moves that include movement in A, B, C axes. 
<pre>
/*   2.1.2.5 Feed Rate
 *
 *  The rate at which the controlled point or the axes move is nominally a steady rate
 *  which may be set by the user. In the Interpreter, the interpretation of the feed
 *  rate is as follows unless inverse time feed rate mode is being used in the
 *  RS274/NGC view (see Section 3.5.19). The canonical machining functions view of feed
 *  rate, as described in Section 4.3.5.1, has conditions under which the set feed rate
 *  is applied differently, but none of these is used in the Interpreter.
 *
 *    A. For motion involving one or more of the X, Y, and Z axes (with or without
 *       simultaneous rotational axis motion), the feed rate means length units per
 *       minute along the programmed XYZ path, as if the rotational axes were not moving.
 *
 *    B. For motion of one rotational axis with X, Y, and Z axes not moving, the
 *       feed rate means degrees per minute rotation of the rotational axis.
 *
 *    C. For motion of two or three rotational axes with X, Y, and Z axes not moving,
 *       the rate is applied as follows. Let dA, dB, and dC be the angles in degrees
 *       through which the A, B, and C axes, respectively, must move.
 *       Let D = sqrt(dA^2 + dB^2 + dC^2). Conceptually, D is a measure of total
 *       angular motion, using the usual Euclidean metric. Let T be the amount of
 *       time required to move through D degrees at the current feed rate in degrees
 *       per minute. The rotational axes should be moved in coordinated linear motion
 *       so that the elapsed time from the start to the end of the motion is T plus
 *       any time required for acceleration or deceleration.
 */ 
</pre>

Notes:
- When computing feed rate for a move, if one or more axis (linear or rotary) is not capable of meeting the acceleration/velocity to achieve the desired feed rate the move will be slowed down to accommodate the rate limiting axis, and hence a lower feed rate will be achieved. 
- In case [A] above, if one or more rotary axes are involved in the move they will complete their motion(s) in the same time that the linear axes are moving. It is possible that the rotary axis (axes) may rate limit the move as described above.
- For g2core builds that support UVW axes g2core extends the NIST definition to include all linear axes (XYZ & UVW) when calculating feed rates.  

## G2, G3 Arc At Feed Rate
See [G2, G3 Arc At Feed Rate](G2-G3-Arc-At-Feed-Rate)

## G4 Dwell
`G4 Pnnn` causes motion to stop and resume nnnn seconds later

- The P number is a required floating point number; fractions of a second may be used
- `G4` affects spindle, coolant and I/O as programmed (details of this are currently incomplete and not listed)

## G5 Reserved for Spline Motion
Currently spline motion is not supported.


## G10 Offsets
See [G10 coordinate offsets](Gcode-Coordinate-Offsets#g10-ln-set-parameters)

## G17, G18, G19 Select Arc Plane
`G17` select XY arc plane
`G18` select XZ arc plane
`G19` select YZ arc plane

## G20, G21 Select Units Mode
`G20` selects inches mode<br>
`G21` selects millimeter mode<br>

## G28, G30 Go To Predefined Position
`G28.1` save location of current point in absolute coordinates<br>
`G28` return to point saved in g28.1<br>
`G28 axes` return to point saved in g28.1 through intermediate point in axes<br><br>
`G30.1` save location of current point in absolute coordinates<br>
`G30` return to point saved in g30.1<br>
`G30 axes` return to point saved in g30.1 through intermediate point in axes<br>

- G28 (and G30) will move the machine to absolute coordinates set in G28.1 (G30.1) at traverse rate (G0)
- If G28.1 or G30.1 is not set the return point is (0,0,0,0,0,0)
- If 'axes' are provided the move will proceed through the intermediate point specified
- Axes that are not specified in the G28 or G30 command are not included in the intermediate move
- Movement will occur in the machine coordinate system (absolute coordinates, G53), and is not affected by work coordinate systems (G54-G59), temporary offsets (G92), or tool length offsets. However, position reporting `{pos:n}` will be provided using all offsets in effect at the time of the G28/G30 command.

Example of use:

- Go to an arbitrary position, e.g. G0 x100 y100
- Send G28.1  - This will "remember" the absolute position. This position remains constant regardless of what coordinate system is in effect.
- Then go to a different place, e.g. G0 x50 y50
- Send G28  - The machine will return to x100 y100

Example:
- Go back to X0Y0
- Send G91 G28 Z10 - this will move to x100 y100. The tool will initially lift z by 10 mm (or inches); G91 is used to set relative mode for this command.

## G38.x Probes
See [G38.x Probes](Gcode-Probes#g38x-probe)

## G53, G54, G55, G56, G57, G58, G59 Coordinate systems
See [Coordinate Systems](Coordinate-Systems).

## G61, G64 Path Control Modes
`G61` set exact path mode<br>
`G61.1` set exact stop mode<br>
`G64` set continuous mode (behaves like exact path mode)<br>

G2core supports exact path mode (G61) and exact stop mode (G61.1). G64 is recognized, but is treated as exact path mode. In exact stop mode motion will stop between each Gcode block. In exact path mode the exact path is followed (i.e. corners are not rounded). The velocity at the points joining 2 blocks is controlled to keep the change in direction between the blocks within the jerk limits of the participating axes.


## G80 Cancel Motion Mode
`G80` cancel motion mode<br>

G80 cancels the current motion mode. Send G80 to make sure no movement will occur. Motion modes are the [NIST Gcode](http://ws680.nist.gov/publication/get_pdf.cfm?pub_id=823374) modal group 1, which includes: G0, G1, G2, G3, G38.2, G80, G81, G82, G83, G84, G85, G86, G87, G88, G89

## G90, G91 Set Distance Mode
`G90` set absolute distance mode (default)<br>
`G91` set incremental distance mode<br>
`G90.1` set absolute arc distance mode<br>
`G91.1` set incremental arc distance mode (default)<br>

In absolute distance mode, axis positions are provided as absolute coordinates in the currently active coordinate system. In incremental distance mode, axis positions represent incremental movement from the current point.

## G92, G92.x
See [Coordinate Systems](Coordinate-Systems).

## G93, G94, G95 Feed Rate Mode
`G93` set inverse-time feedrate mode<br>
`G94` set units-per-minutes feedrate mode<br>
`G95` reserved for set units-per-revolution feedrate mode (unimplemented)<br>
