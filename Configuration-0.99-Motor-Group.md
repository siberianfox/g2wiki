
##Motor Groups
Settings specific to a given motor. There are 6 motor groups, numbered 1,2,3,4,5,6. These are labeled on the v9 board, which breaks out motor1 - motor4. Other platforms may make more or fewer motors available, e.g. the Due has outputs for all 6 motors, which are labeled [here](Arduino-DUE-Pinout-for-tinyG2).<br><br>

	Setting | Description | Notes
	--------|-------------|-----------------------------
	[{1ma:_}](#1ma---map-motor-to-axis) | Map to Axis | Configure axis to which this motor is connected (for Cartesian machines) E.g. {1ma:0}, {2ma:1}, {3ma:2}, {4ma:3} to map motors 1-4 to X,Y,Z,A, respectively
	[{1sa:_}](#1sa---step-angle-for-the-motor) | Step Angle | Motor parameter indicating the angle traveled per whole step. Typical setting is {1sa:1.8} for 1.8 degrees per step (200 steps per revolution)
	[{1tr:_}](#1tr---travel-per-revolution) | Travel_per_Rev | How far the mapped axis moves per motor revolution. E.g. {1tr:2.54} (millimeters) for a 10 TPI screw axis
	[{1mi:_}](#1mi---microsteps) | MIcrosteps | Microsteps per whole step. G2 uses 1,2,4,8,16,32. Other values are accepted but warned
	[{1su:_}](#1su---steps-per-unit) | Steps per Unit | Direct input and reading of steps per unit - about 7 digits decimal accuracy
	[{1po:_}](#1po---polarity) | POlarity | Set polarity for proper movement of the axis. 0=clockwise rotation, 1=counterclockwise - although these are dependent on your motor wiring, and axis movement is dependent on the mechanical system.
	[{1pm:_}](Power-Management) | Power Mode | 0=motor disabled, 1=motor always on, 2=motor on when in cycle, 3=motor on only when moving
	[{1pl:_}](#1pl---power-level) | Power Level | 0.000=no power to steppers, 1.000=max power to steppers

_Note: Setting power level to every high values will not necessarily get more power to the motors. At some point you may  saturate the coils, get worse motor performance, draw more current, and risk burning out the motors by overheating._

