OK, There is no "standard" Gcode, despite multiple attempts to establish one. This attached spreadsheet contains a rough consensus for common Gcode and Mcode use derived from the following sources:

- NIST
- LinuxCNC
- Haas 
- Fanuc
- Tormach
- CNC Cookbook

In the table below all the above use this command similarly. Reprap usage is provided in the next table.

	Gcode | Command | Usage / Notes
	--------|-------------|-----------------------------
	G0 | Coordinated Straight Motion Rapid Rate (Rapid Traverse) | 
	G1 | Coordinated Straight Motion at Feed Rate | Feed rate is honored, as are abs/inv-time feed rate modes. F is modal and may be set before or in the Gcode block
	G2 | Clockwise Circular/Helical Interpolation at Feed Rate | Controlled Arc Move
	G3 | Counterclockwise Circular/Helical Interpolation at Feed Rate | Controlled Arc Move
	G4 | Dwell | Dwell is always P in seconds (not milliseconds)
	G5.x | Reserved for curve and spline  interpolation |
	G5 |Cubic Spline | LinuxCNC
	G5.1 |Quadratic B-Spline | LinuxCNC
	G5.2 |NURBS, add control point | LinuxCNC
	G5.3 |NURBS, execute | LinuxCNC
	G6 | Not used |
	G7 | Diameter Mode (lathe) |
	G8 | Radius Mode (lathe) |
	G9 | Exact Stop (non-modal) | Fanuc, Haas
	G10 | Programmable Data Input | See G10 Lxx commands below
	G10 L1 | Set Tool Table Entry 
	G10 L1	Set Tool Table Entry		
	G10 L10	Set Tool Table, Calculated, Workpiece		
	G10 L11	Set Tool Table, Calculated, Fixture		
	G10 L2	Coordinate System Origin Setting
	G10 L20	Coordinate Origin Setting Calculated

	G10	Tool Offset	reprap	Use G43 / G49 instead					G10	Tool Offset (RRF only)
G10	Retract	reprap	Deprecate in favor fo firmware controlled retract					G10	Retract (Marlin, Repetier, Smoothie)
G11	Unretract	reprap	Deprecate in favor fo firmware controlled retract					G11	Unretract (Marlin, Repetier, Smoothie)
G12	<reserved>	Tormach				G12	CW circular pocket (Haas, Tormach)		
G13	<reserved>	Tormach				G13	CCW circular pocket (Haas, Tormach)		
G15	<reserved>	Tormach				G15	Polar coordinates (Tormach, CNC Cookbook)		
G16	<reserved>	Tormach				G16	Polar coordinates (Tormach, CNC Cookbook)		
G17	Select XY Plane	cnc		G17	XY-Plane Selection	G17	Select XY Plane	G17	Plane Selection (CNC specific)
						G17.1	Select UV Plane		
G18	Select XZ Plane	cnc		G18	XZ-Plane Selection	G18	Select XZ Plane	G18	Plane Selection (CNC specific)
						G18.1	Select WU Plane		
G19	Select YZ Plane	cnc		G19	YZ-Plane Selection	G19	Select YZ Plane	G19	Plane Selection (CNC specific)
						G19.1	Select VW Plane		
G20	Inch Units (Imperial)	cnc	Units selection governs movement, displays, and settings	G20	Inch System Selection	G20	Inch Units	G20	Set Units to Inches
G21	Millimeter Units (Metric)	cnc	Units selection governs movement, displays, and settings	G21	Millimeter System Selection	G21	Millimeter Units	G21	Set Units to Millimeters
G22	Firmware Controlled Retract	MachineKit	Up for discussion					G22	Firmware Controlled Retract (MachineKit)
G23	Firmware Controlled Precharge	MachineKit	Up for discussion					G23	Firmware Controlled Precharge (MachineKit)
G27	<reserved>	Fanuc				G27	Reference Position Check (Fanuc)		
G28	Go To Predefined Position Through Point (G28)	nist	Move to G28.1 stored position via optional intermediate point	G28	Return To Home	G28	Go To Predefined Position Through Point (G28)		
G28.1	Set Predefined Position (G28)	cnc	Store current position. All axes are stored			G28.1	Set Predefined Position (G28)	G28	Move to Origin (Homing Cycle)
G28.2	Homing Cycle	tinyg	Move to JSON as it should not be used in a cycle (deprecate)						
G28.3	Set Axis as Homed	tinyg	Move to JSON as it should not be used in a cycle (eliminate)						
G28.4 	Set Axis as Homed to Value	tinyg	Move to JSON as it should not be used in a cycle (eliminate)						
G29	<reserved>	Haas				G29	Go to G29 Reference Point (Haas)		
G29	Detailed Z-Probe	Ma, MK	Why are these moves in G29 and not G38? USE JSON INSTEAD					G29	Detailed Z-Probe (Marlin, MachineKit)
G29.1	Set Z probe head offset	MachineKit	Try to find a way to extend G38.x for these specialized probes					G29.1	Set Z probe head offset (MachineKit)
G29.2	Set Z probe head offset calculated from toolhead position	MachineKit						G29.2	Set Z probe head offset calculated from toolhead position (MK)
G30	Single Z probe							G30	Single Z-Probe (Ma, Re, Sm, RRF)
G30	Go To Predefined Position Through Point (G30)	nist	Move to G28.1 stored position via optional intermediate point	G30	Return To Secondary Home G38.2	G30	Go To Predefined Position Through Point (G30)		
G30.1	Set Predefined Position (G30)	cnc	Store current position. All axes are stored			G30.1	Set Predefined Position (G30)		
G31	<reserved>	Haas				G31	Straight Probe Until Skip (Haas, Tormach)		
G31	Report Current Probe status	Sm, RRF	Try to find a way to extend G38.x for these specialized probes					G31	Report Current Probe Status (Sm, RRF)
G32	Probe Z and calculate Z plane	Sm, RRF						G32	Probe Z and calculate Z plane (Sm, RRF)
G31	Dock Z Probe sled	Marlin						G31	Dock Z Probe sled (Marlin)
G32	Undock Z Probe sled	Marlin						G32	Undock Z Probe sled (Marlin)
						G32	Thread Cutting (Fanuc)		
						G33	Spindle Synchronized Motion		
						G33.1	Rigid Tapping		
G35	<reserved>	Haas				G35	Automatic Tool Diameter Measurement (Haas)		
G36	<reserved>	Haas				G36	Automatic Work Offset Measurement (Haas)		
G37	<reserved>	Haas				G37	Automatic Tool Length Measurement (Haas)		
G38.2	Probe To Workpiece, Error If Failure	cnc		G38.2	Straight Probe	G38.2	Probe To Workpiece, Error If Failure	G38.2	probe toward workpiece
G38.3	Probe To Workpiece, No Error If Failure	cnc				G38.3	Probe To Workpiece, No Error If Failure	G38.3	probe toward workpiece
G38.4	Probe Away From Workpiece, Error If Failure	cnc				G38.4	Probe Away From Workpiece, Error If Failure	G38.4	probe away from workpiece
G38.5	Probe Away From Workpiece, No Error If Failure	cnc				G38.5	Probe Away From Workpiece, No Error If Failure	G38.5	probe away from workpiece
G40	<reserved>	cnc		G40	Cancel Cutter Radius Compensation	G40	Cancel Cutter Compensation	G40	Compensation Off (CNC specific)
G41	<reserved>	cnc		G41	Start Cutter Radius Compensation Left	G41	Cutter Compensation, Left		
						G41.1	Dynamic Cutter Compensation		
G42	<reserved>	cnc		G42	Start Cutter Radius Compensation Right	G42	Cutter Compensation, Right		
						G42.1	Dynamic Cutter Compensation		
G43	Tool Length Compensation	cnc		G43	Tool Length Offset (plus)	G43	Use Tool Length Offset from Tool Table		
						G43.1	Dynamic Tool Length Offset		
						G43.2	Apply additional Tool Length Offset		
						G44	Tool Length Compensation, Negative (Fanuc, Haas)		
G49	Cancel Tool Length Compensation	cnc		G49	Cancel Tool Length Offset	G49	Cancel Tool Length Compensation		
G50	<reserved>	Haas				G50	Reset Scale Factors to 1.0 (Haas, Tormach)		
G51	<reserved>	Haas				G51	Set Axis Data Input Scale Factors (Haas, Tormach)		
G51	<reserved>	Haas				G52	Local Work Shift (Fanuc, Haas)		
G53	Machine Coordinate System (Non-Modal)	cnc		G53	Motion In Machine Coordinate System	G53	Machine Coordinate System (Non-Modal)		
G54	Select Coordinate System 1	cnc		G54	Use Preset Work Coordinate System 1	G54	Select Coordinate System 1	G54	Coordinate System Select (CNC specific)
G55	Select Coordinate System 2	cnc		G55	Use Preset Work Coordinate System 2	G55	Select Coordinate System 2	G55	Coordinate System Select (CNC specific)
G56	Select Coordinate System 3	cnc		G56	Use Preset Work Coordinate System 3	G56	Select Coordinate System 3	G56	Coordinate System Select (CNC specific)
G57	Select Coordinate System 4	cnc		G57	Use Preset Work Coordinate System 4	G57	Select Coordinate System 4	G57	Coordinate System Select (CNC specific)
G58	Select Coordinate System 5	cnc		G58	Use Preset Work Coordinate System 5	G58	Select Coordinate System 5	G58	Coordinate System Select (CNC specific)
G59	Select Coordinate System 6	cnc		G59	Use Preset Work Coordinate System 6	G59	Select Coordinate System 6	G59	Coordinate System Select (CNC specific)
G59.1	<reserved>	cnc		G59.1	Use Preset Work Coordinate System 7	G59.1	Select Coordinate System 7		
G59.2	<reserved>	cnc		G59.2	Use Preset Work Coordinate System 8	G59.2	Select Coordinate System 8		
G59.3	<reserved>	cnc		G59.3	Use Preset Work Coordinate System 9	G59.3	Select Coordinate System 9		
						G60	Unidirectional Positioning (Haas)		
G61	Exact Path Mode	nist		G61	Exact Path Mode	G61	Exact Path Mode		
G61.1	Exact Stop Mode	nist		G61.1	Exact Stop Mode	G61.1	Exact Stop Mode		
						G62	Automatic Corner Override (CNC Cookbook)		
						G63	Tapping Mode (CNC Cookbook)		
G64	Continuous Mode	nist		G64	Continuous Mode	G64	Path Blending Mode		
						G65	Macro Subroutine Call (Haas)		
G68	<reserved>	Fanuc				G68	Coordinate System Rotation (All Mfrs)		
G69	<reserved>	Fanuc				G69	Cancel Coordinate System Rotation (All Mfrs)		
						G70	Bolt Hole Circle (Haas)		
						G71	Bolt Hole Arc (Haas)		
						G72	Bolt Holes Along and Angle (Haas)		
						G73	Drilling Cycle with Chip Breaking		
						G74	Reverse Tap Canned Cycle (Haas)		
						G76	Multi-pass Threading Cycle (Lathe)		
						G77	Back Bore Canned Cycle (Haas)		
G80	Cancel Motion Modes	nist		G80	Cancel Motion Mode (including Canned Cycle)	G80	Cancel Motion Modes	G80	Cancel Canned Cycle (CNC specific)
G81	<reserved>	cnc		G81	Canned Cycle: drilling	G81	Drilling Cycle		
G82	<reserved>	cnc		G82	Canned Cycle: drilling with dwell	G82	Drilling Cycle with Dwell		
G83	<reserved>	cnc		G83	Canned Cycle: peck drilling	G83	Drilling Cycle with Peck		
G84	<reserved>	cnc		G84	Canned Cycle: right hand tapping	G84	Tapping Canned Cycle (Haas)		
G85	<reserved>	cnc		G85	Canned Cycle: boring	G85	Boring Cycle, No Dwell, Feed Out		
G86	<reserved>	cnc		G86	Canned Cycle: boring	G86	Boring Cycle, Stop, Rapid Out		
G87	<reserved>	cnc		G87	Canned Cycle: back boring	G87	Bore/Manual Retract Canned Cycle (Haas)		
G88	<reserved>	cnc		G88	Canned Cycle: boring	G88	Bore/Dwell Canned Cycle (Haas)		
G89	<reserved>	cnc		G89	Canned Cycle: boring	G89	Boring Cycle, Dwell, Feed Out		
G90	Absolute Distance Mode	cnc		G90	Absolute Distance Mode	G90	Absolute Distance Mode	G90	Set to Absolute Positioning
G09.1	Absolute Arc Distance Mode	cnc				G09.1	Absolute Arc Distance Mode		
G91	Incremental Distance Mode	cnc		G91	Incremental Distance Mode	G91	Incremental Distance Mode	G91	Set to Relative Positioning
G91.1	Incremental Arc Distance Mode	cnc				G91.1	Incremental Arc Distance Mode		
								G91.x	Reset Coordinate System Offsets (CNC specific)
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