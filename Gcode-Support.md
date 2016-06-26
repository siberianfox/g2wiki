**NOTE: This page describes Gcode supported by G2 in firmware builds 083.xx and later**

G2 implements the NIST RS274v3/ngc dialect of Gcode informed by the LinuxCNC Gcode specification:
* [Kramer's NIST RS274NGCv3 Gcode Specification](http://technisoftdirect.com/catalog/download/RS274NGC_3.pdf)
* [LinuxCNC Gcode Specification](http://www.linuxcnc.org/docs/2.4/html/gcode_main.html)<br>

##Cheat Sheet
This table summarizes Gcode supported. _axes_ means one or more of X,Y,Z,A,B,C. 

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G0 | _axes_ | Straight traverse | Traverse at maximum velocity
	G1 | _axes_, F | Straight feed | Move at feed rate F
	G2 | _axes_, F, I,J,K or R | Clockwise arc feed | Arc at feed rate F. Offset mode IJK or radius mode R
	G3 | _axes_, F, I,J,K or R | Counter clockwise arc feed | Arc at feed rate F. Offset mode IJK or radius mode R
	G4 | P | Dwell | Pause for P seconds
	[G10 L2](#g10-l2-set-parameters-offsets) | _axes_, P | Set offset parameters | P selects coordinate system 1-6
	[G10 L20](#g10-l20-set-parameters-offsets) | _axes_, P | Set offset parameters | P selects coordinate system 1-6
	G17 | | Select XY plane | G17, G18 and G19 set the plane in which the G2/G3 arcs are drawn
	G18 | | Select XZ plane |
	G19 | | Select YZ plane |
	[G20](Inch-and-Millimeter-Units-Mode) | | Select inches units mode | All Gcode from this point on will be interpreted in inches
	[G21](Inch-and-Millimeter-Units-Mode) | | Select mm units mode | All Gcode from this point on will be interpreted in millimeters
	[G28](#g28-g281-g30-g301-go-to-predefined-position) | _axes_ | Go to G28.1 position | Optional axes specify an intermediate point
	[G28.1](#g28-g281-g30-g301-go-to-predefined-position) | | Set position for G28 | The current machine position is recorded (No parameters are provided)
	[G28.2](https://github.com/synthetos/TinyG/wiki/Homing-and-Limits-Description-and-Operation#g282---homing-sequence-homing-cycle) | _axes_ | Homing Sequence | Homes all axes present in command. At least one axis must be specified
	[G28.3](https://github.com/synthetos/TinyG/wiki/Homing-and-Limits-Description-and-Operation#g283---set-absolute-position) | _axes_ | Set Absolute Position | Set axis to zero or other value. Use to zero axes that cannot otherwise be homed
	[G30](#g28-g281-g30-g301-go-to-predefined-position) | _axes_ | Go to G30.1 position | Optional axes specify an intermediate point
	[G30.1](#g28-g281-g30-g301-go-to-predefined-position) | | Set position for G30 | The current machine position is recorded (No parameters are provided)
	G38.2 | Probe toward, do not report error | Supported in Build 082.xx
	G38.3 | Probe toward, report if error | Supported in Build 100.xx
	G38.4 | Probe away, do not report error | Supported in Build 100.xx 
	G38.5 | Probe away, report if error | Supported in Build 100.xx
	G53 | | Select absolute coordinates | Non-Modal: Applies only to current block
	G54 | | Select coord system 1 | G54 is typically used as the "normal" coordinate system and reflects the machine position
	G55 | | Select coord system 2 |
	G56 | | Select coord system 3 |
	G57 | | Select coord system 4 |
	G58 | | Select coord system 5 |
	G59 | | Select coord system 6 |
	[G61](#g61-g611-g64-path-control-modes) | | Exact stop mode | Motion will stop between each Gcode block
	[G61.1](#g61-g611-g64-path-control-modes) | | Exact path mode | Continuous motion between Gcode blocks - exact path will be traced
	[G64](#g61-g611-g64-path-control-modes) | | Continuous path mode | Same as exact path mode 
	G80 | | Cancel motion mode |
	G90 | | Set absolute distance mode |
	G90.1 | | Set absolute arc distance mode |
	G91 | | Set incremental distance mode |
	G91.1 | | Set incremental arc distance mode |
	G92 | _axes_ | Set origin offsets |
	G92.1 | | Reset origin offsets |
	G92.2 | | Suspend origin offsets |
	G92.3 | | Resume origin offsets |
	G93 | | Set inverse feedrate mode |
	G94 | | Cancel inverse feedrate mode |

 	Mcode | Parameter |Command | Description
	------|-----------|--------|-------------
	M0 | | Program stop |
	M1 | | Program stop | Optional program stop switch is not implemented so M1 is equivalent to M0
	[M2](#m2-m30-program-end) | | Program end |
	[M30](#m2-m30-program-end) | | Program end |
	M60 | | Program stop |
	M3 | S | Spindle on - CW | S is speed in RPM
	M4 | S | Spindle on - CCW | S is speed in RPM
	M5 | | Spindle off |
	M6 | | Change tool | No operation at this time
	M7 | | Mist coolant on | Note that mist and flood share the same Coolant ON/OFF pin
	M8 | | Flood coolant on | Note that mist and flood share the same Coolant ON/OFF pin
	M9 | | All coolant off | Note that mist and flood share the same Coolant ON/OFF pin

 	Other | Parameter |Command | Description
	------|-----------|--------|-------------
	N | line number | label gcode block | Line numbers are allowed, handled, and may be reported back in status reports. Don't underestimate how useful this is for debugging Gcode files.
	[()](#gcode-comments) | [comment](#gcode-comments) | [gcode comment](#gcode-comments) | Gcode comments are supported. They are stripped and ignored, except for messages (below)
	[;](#gcode-comments) | comment | alternate comment | A semicolon is an alternate way to delimit a comment. This is not Gcode "standard", but is used by Mach and some Reprap codes. (available as of build 378.05)
	(msg....) | message | gcode message | Gcode messages are comments that begin with the characters `msg` (case insensitive). These will be echoed to the operator 


#Gcode Reference
Some of the more complicated commands are described here. Much of this is shamelessly cribbed from the [LinuxCNC Gcode pages](http://www.linuxcnc.org/docs/2.4/html/gcode_main.html)<br>

##G10 L2 Set Parameters (Offsets)
The G10 L2 command is used to set coordinate offsets. Use Pn to select coordinate system 1-6 (G54 - G59, respectively), and one or more axis values to set the offset for that axis. The axis value is the offset you want for that axis, relative the zero set during homing. For example, if you wanted the X origin of coordinate system 1 (G54) to be 100 mm to the right of zero you would enter G10 L2 X100.

##G10 L20 Set Parameters (Offsets)
The G10 L20 command is also used to set coordinate offsets. Use Pn to select coordinate system 1-6 (G54 - G59, respectively), and one or more axis values to set the offset for that axis. The axis value is the value you wish the current point to be. For example, if you wanted the current X location to be X0 in coordinate system 1 (G54) you would enter G10 L20 X0.

##G28, G28.1, G30, G30.1 Go To Predefined Position
G28 (and G30) will move the machine to the absolute coordinates set in G28.1 (G30.1), optionally through an intermediate point.  Movement will occur at the traverse rate (G0 rate). Axes that are not specified are ignored (not moved). The axis value is the intermediate point for that axis. 

Format is:
<pre>G28 X0 Y0 Z0 A0 B0 C0</pre>

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G28 | _axes_ | Go to G28.1 position | Goes through intermediate point if _axes_ are present
	G28.1 | | Set position for G28 | Axes are not used and are ignored if present
	G30 | _axes_ | Go to G30.1 position | Goes through intermediate point if _axes_ are present
	G30.1 | | Set position for G30 | Axes are not used and are ignored if present

Example of use: 
* Go to an arbitrary position, e.g. G0 x100 y100 
* Send G28.1  - This will "remember" the absolute position. This position remains constant regardless of what coordinate system is in effect. 
* Then go to a different place, e.g. G0 x50 y50
* Send G28  - The machine will return to x100 y100

Example:
* Go back to X0Y0
* Send G91 G28 Z10 - this will move to x100 y100. The tool will initially lift z by 10 mm (or inches); G91 is used to set relative mode for this command. 

##G61, G61.1, G64 Path Control Modes
TinyG supports exact stop mode (G61) and exact path mode (G61.1). G64 is recognized, but is treated as exact path mode. In exact stop mode motion will stop between each Gcode block. In exact path mode the exact path is followed (i.e. corners are not rounded). The velocity at the points joining 2 blocks is controlled to keep the change in direction between the blocks within centripetal acceleration limit set by $JA. Please see [here](https://github.com/synthetos/TinyG/wiki/Configuration#xjd---junction-deviation) for an explanation.

##M2, M30 Program End
program END (M2, M30) performs the following actions:

* Axis offsets are set to G92.1 CANCEL offsets
* Set default coordinate system (uses $gco)
* Selected plane is set to default plane ($gpl)
* Distance mode is set to MODE_ABSOLUTE (like G90)
* Feed rate mode is set to UNITS_PER_MINUTE (like G94)
* The spindle is stopped (like M5)
* Motion mode is canceled like G80 (not set to G1)
* Coolant is turned off (like M9)
* Default INCHES or MM units mode is restored ($gun)

##Gcode Comments
 * Comments start with a '(' char or alternately a semicolon ';' 
 * Comments always terminate the block - i.e. leading or embedded comments are not supported
<pre>
Valid comment cases       Notes:
G0X10                      - command only - no comment
G0X10 (comment text)       - comment with comment
G0X10 (comment text        - it's OK to drop the trailing paren
G0X10 ;comment text        - comment delimited by semicolon (firmware build 378.05 and later) 
(comment text)             - there is no command on this line
</pre>
<pre>
Invalid comment cases     Notes:
G0X10 comment text         - comment with no separator
N10 (comment) G0X10        - embedded comment. G0X10 will be ignored
(comment) G0X10            - leading comment. G0X10 will be ignored
G0X10 #comment             - invalid comment separator
</pre>