OK, There is no "standard" Gcode, despite multiple attempts to establish one. This page collects common Gcode and Mcode uses derived from the following sources:

- NIST
- LinuxCNC
- Haas 
- Fanuc
- Tormach
- CNC Cookbook

It also lists Reprap, Machinekit, TinyG and other usage that is incompatible with the common usage, and provides some notes and some recommendations for alternatives.

##Consensus Gcode Usage
This table lists rough consensus usage from the above sources. 

	Gcode | Command | Usage / Notes
	--------|-------------|-----------------------------
	G0 | Coordinated Straight Motion at Rapid Rate | Rapid Traverse
	G1 | Coordinated Straight Motion at Feed Rate | Feed rate is honored, as are abs/inv-time feed rate modes
	G2 | Clockwise Circular/Helical Interpolation at Feed Rate | Controlled Arc Move
	G3 | Counterclockwise Circular/Helical Interpolation at Feed Rate | Controlled Arc Move
	G4 | Dwell | P is in seconds, not milliseconds or other units
	G5.x | Reserved for curve and spline  interpolation |
	G5 | Cubic Spline | (LinuxCNC)
	G5.1 |Quadratic B-Spline | (LinuxCNC)
	G5.2 |NURBS, add control point | (LinuxCNC)
	G5.3 |NURBS, execute | (LinuxCNC)
	G6 | Not used |
	G7 | Diameter Mode | Lathe usage
	G8 | Radius Mode | Lathe usage
	G9 | Exact Stop (non-modal) | Fanuc, Haas
	G10 | Programmable Data Input | See G10 Lxx commands below
	G10 L1 | Set Tool Table Entry |
	G10 L10 | Set Tool Table, Calculated, Workpiece |
	G10 L11 | Set Tool Table, Calculated, Fixture |
	G10 L2 | Coordinate System Origin Setting |
	G10 L20 | Coordinate Origin Setting Calculated |
	G11 | Not Used |
	G12 | CW circular pocket | (Haas, Tormach)
	G13 | CCW circular pocket | (Haas, Tormach) 
	G15 | Polar coordinates | (Tormach, CNC Cookbook)
	G16 | Polar coordinates | (Tormach, CNC Cookbook) 
	G17 | Select XY Plane |
	G17.1 | Select UV Plane | 
	G18 | Select XZ Plane |
	G18.1 | Select UW Plane | 
	G19 | Select YZ Plane |
	G19.1 | Select VW Plane | 
	G20 | Set Units to Inches (Imperial) | Units selection governs movement, displays, and settings
	G21 | Set Units to Millimeters (Metric) | Units selection governs movement, displays, and settings
	G22 | Not used |
	G23 | Not used |
	G24 | Not used |
	G25 | Not used |
	G26 | Not used |
	G27 | Reference Position Check | (Fanuc)	
	G28 | Go To Predefined Position Through Point (G28) | Move to G28.1 stored position via optional intermediate point
	G28.1 | Set Predefined Position | Store current position for G28. All axes are stored.
	G29 |  Go to G29 Reference Point | (Haas)
	G30 | Go To Predefined Position Through Point (G30) | Move to G30.1 stored position via optional intermediate point
	G30.1 | Set Predefined Position | Store current position for G30. All axes are stored.
	G31 | Straight Probe Until Skip | (Haas, Tormach)
	G32 | Thread Cutting | (Fanuc)
	G33 | Spindle Synchronized Motion
	G33.1 | Rigid Tapping
	G34 | Not used
	G35 | Automatic Tool Diameter Measurement | (Haas)
	G36 | Automatic Work Offset Measurement | (Haas)
	G37 | Automatic Tool Length Measurement | (Haas)
	G38.2 | Straight Probe To Workpiece, Report if failure |
	G38.3 | Straight Probe To Workpiece |
	G38.4 | Straight Probe Away From Workpiece, Report if failure |
	G38.5 | Straight Probe Away From Workpiece |
	G39 | Not used |
	G40 | Cancel Cutter Compensation | Turn Compensation Off
	G41 | Start Cutter Radius Compensation Left |
	G41.1 | Dynamic Cutter Compensation |
	G42 | Start Cutter Radius Compensation Right |
	G42.1 | Dynamic Cutter Compensation |
	G43 | Tool Length Offset | Use Tool Length Offset from Tool Table. 
	G43 | Tool Length Compensation, Positive | (Fanuc, Haas)	
	G43.1 | Dynamic Tool Length Offset |
	G43.2 | Apply additional Tool Length Offset |	
	G44 | Tool Length Compensation, Negative (Fanuc, Haas) |
	G49 | Cancel Tool Length Compensation |
	G50 | Reset Scale Factors to 1.0 | (Haas, Tormach)
	G51 | Set Axis Data Input Scale Factors | (Haas, Tormach)
	G52 | Local Work Shift | (Fanuc, Haas)
	G53 | Motion In Machine Coordinate System | Non-Modal
	G54 | Select Coordinate System 1 | Use Preset Work Coordinate System 1
	G55 | Select Coordinate System 2 | Use Preset Work Coordinate System 2
	G56 | Select Coordinate System 3 | Use Preset Work Coordinate System 3
	G57 | Select Coordinate System 4 | Use Preset Work Coordinate System 4
	G58 | Select Coordinate System 5 | Use Preset Work Coordinate System 5
	G59 | Select Coordinate System 6 | Use Preset Work Coordinate System 6
	G59.1 | Select Coordinate System 7 | Use Preset Work Coordinate System 7
	G59.2 | Select Coordinate System 8 | Use Preset Work Coordinate System 8
	G59.3 | Select Coordinate System 9 | Use Preset Work Coordinate System 9
	G60 | Unidirectional Positioning | (Haas)
	G61 | Exact Path Mode |
	G61.1 | Exact Stop Mode	|
	G62 | Automatic Corner Override | (CNC Cookbook)
	G63 | Tapping Mode | (CNC Cookbook)
	G64 | Continuous Mode | Path Blending Mode
	G65 | Macro Subroutine Call | (Haas)
	G68 | Coordinate System Rotation | 
	G69 | Cancel Coordinate System Rotation |
	G70 | Bolt Hole Circle | (Haas)
	G71 | Bolt Hole Arc | (Haas)
	G72 | Bolt Holes Along and Angle | (Haas)	
	G73 | Drilling Cycle with Chip Breaking
	G74 | Reverse Tap Canned Cycle | (Haas)
	G76 | Multi-pass Threading Cycle | (Lathe)
	G77 | Back Bore Canned Cycle | (Haas)
	G80 | Cancel Motion Mode | including Canned Cycle
	G81 | Drilling Cycle |
	G82 | Drilling Cycle with Dwell |
	G83 | Drilling Cycle with Peck |
	G84 | Tapping Canned Cycle | (Haas)	
	G85 | Boring Cycle, No Dwell, Feed Out
	G86 | Boring Cycle, Stop, Rapid Out
	G87 | Bore/Manual Retract Canned Cycle | (Haas)
	G88 | Bore/Dwell Canned Cycle | (Haas)
	G89 | Boring Cycle, Dwell, Feed Out |
	G90 | Absolute Distance Mode |
	G09.1 | Absolute Arc Distance Mode |
	G91 | Incremental Distance Mode	| Set to Relative Positioning
	G91.1 |Incremental Arc Distance Mode |
	G91.x | Reset Coordinate System Offsets | 

G92	Offset Coordinate Systems & Set Parameters	nist		G92	Offset Coordinate Systems & Set Parameters	G92	Coordinate System Offset	G92	Set Position
G92.1	Cancel Offset Coordinate Systems & Set Parameters	nist		G92.1	Cancel Offset Coordinate Systems & Set Parameters	G92.1	Cancel Coordinate System Offsets		
G92.2	Cancel Offset Coordinate Systems, Do Not Reset Parameters	nist	Optional	G92.2	Cancel Offset Coordinate Systems, Do Not Reset Parameters	G92.2			
G92.3	Apply Parameters to Offset Coordinate Systems	nist	Optional	G92.3	Apply Parameters to Offset Coordinate Systems	G92.3	Restore Axis Offsets		
G93	Inverse Time Feed Rate Mode	cnc		G93	Inverse Time Feed Rate Mode	G93	Inverse Time Feed Rate Mode	G93	Feed Rate Mode (Inverse Time Mode) (CNC specific)
G94	Units Per Minute Feed Rate Mode	cnc		G94	Units Per Minute Feed Rate Mode	G94	Units Per Minute Feed Rate Mode	G94	Feed Rate Mode (Units per Minute) (CNC specific)
G95	<reserved>					G95	Units Per Revolution Feed Rate Mode		
G96	<reserved>					G96	Constant Surface Speed		
						G97	RPM Mode (Cancel Constant Surface Speed)		
G98	<reserved>	cnc		G98	Initial Level Return In Canned Cycles	G98	Canned Cycle Z Retract Mode		
G99	<reserved>	cnc		G99	R-point Level Return In Canned Cycles	G99	R-point Level Return In Canned Cycles		
						G100 -G188	Haas Gcodes		
G100	Calibrate floor or rod radius	Reptitier	Do not implement					G100	Calibrate floor or rod radius
G130	Set digital potentiometer value	Makerbot	Do not implement					G130	Set digital potentiometer value
G131	Remove offset	Reptitier	Do not implement					G131	Remove offset
G132	Calibrate endstop offsets	Reptitier	Do not implement					G132	Calibrate endstop offsets
G133	Measure steps to top	Reptitier	Do not implement					G133	Measure steps to top
G161	Home axes to minimum	Makerbot	Remove - Use G28 homing instead					G161	Home axes to minimum
G162	Home axes to maximum	Makerbot	Remove - Use G28 homing instead					G162	Home axes to maximum

##Exceptions to Consensus Gcode Usage
