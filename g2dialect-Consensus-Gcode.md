This page provides reference information used by the [g2dialect](g2dialect).

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
	G10_L20 | Coordinate Origin Setting Calculated |
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
	G92 | Set Coordinate System Offsets |
	G92.1 | Cancel Coordinate System Offsets |
	G92.2 | Cancel Offset Coordinate Systems, Do Not Reset Parameters
	G92.3 | Apply Parameters to Offset Coordinate Systems | Restore Axis Offsets	
	G93 | Inverse Time Feed Rate Mode | Inverse Time Mode
	G94 | Units Per Minute Feed Rate Mode | Feed Rate Mode
	G95 | Units Per Revolution Feed Rate Mode |
	G96 | Constant Surface Speed |
	G97 | RPM Mode | Cancel Constant Surface Speed
	G98 | Initial Level Return In Canned Cycles | Canned Cycle Z Retract Mode
	G99 | R-point Level Return In Canned Cycles |
	G100+ | Haas Gcodes continue from G100 to G188 |

##Exceptions to Consensus Gcode Usage
The following table lists incompatibilities **(bolded)** with consensus Gcode. Incompatibilities may be due to:

- Differences in implementation from a consensus Gcode command
- Differences in parameter usage from a consensus Gcode command
- Additional or incompatible dot extensions
- Additional Gcode commands that are not in the consensus set

The implementation is noted in (Parens). When (Reprap) is noted it means that one or more of the major Reprap implementations do this, as there are variations. 

	Gcode | Consensus Usage | Non-Consensus Usage / Notes
	--------|-------------|-----------------------------
	G0 | Coordinated Straight Motion at Rapid Rate | **(Reprap) provides feed rate for G0. (Reprap) uses S to set endstop options during movement, (Reprap) defines E axes, which are not part of the Gcode axis set (XYZ ABC UVW). (Reprap) may invoke retraction and recharge on G0.**
	G1 | Coordinated Straight Motion at Feed Rate | **(Reprap) uses S to set endstop options during movement, (Reprap) defines E axes, which are not part of the Gcode axis set (XYZ ABC UVW)**
	G2 | Clockwise Circular/Helical Interpolation at Feed Rate | **(Reprap) motion features similar to G1.** Note: circular/helical motion is rarely used in 3D printing.
	G3 | Counterclockwise Circular/Helical Interpolation at Feed Rate | **(Reprap) motion features similar to G1.** Note: circular/helical motion is rarely used in 3D printing.
	G4 | Dwell | **(Reprap) dwell may use S to set dwell time in milliseconds (not seconds). Uses P for seconds.** Note: S is a modal word who's usage here is incompatible as it conflicts with Spindle RPM setting
	G5.x | Reserved for curve and spline  interpolation |
	G5 | Cubic Spline |
	G5.1 |Quadratic B-Spline |
	G5.2 |NURBS, add control point |
	G5.3 |NURBS, execute |
	G6 | Not used |
	G7 | Diameter Mode |
	G8 | Radius Mode |
	G9 | Exact Stop (non-modal) |
	G10 | Programmable Data Input |
	G10 L1 | Set Tool Table Entry |
	G10 L10 | Set Tool Table, Calculated, Workpiece |
	G10 L11 | Set Tool Table, Calculated, Fixture |
	G10 L2 | Coordinate System Origin Setting |
	G10_L20 | Coordinate Origin Setting Calculated |
	G11 | Not Used |
	G12 | CW circular pocket |
	G13 | CCW circular pocket |
	G15 | Polar coordinates |
	G16 | Polar coordinates |
	G17 | Select XY Plane |
	G17.1 | Select UV Plane | 
	G18 | Select XZ Plane |
	G18.1 | Select UW Plane | 
	G19 | Select YZ Plane |
	G19.1 | Select VW Plane | 
	G20 | Set Units to Inches (Imperial) |
	G21 | Set Units to Millimeters (Metric) |
	G22 | Not used | **(MachineKit) Firmware Controlled Retract**
	G23 | Not used | **(MachineKit) Firmware Controlled Precharge**
	G24 | Not used |
	G25 | Not used |
	G26 | Not used |
	G27 | Reference Position Check |	
	G28 | Go To Predefined Position Through Point (G28) |
	G28.1 | Set Predefined Position |
	G28.2 | Not used | **(TinyG) Homing Sequence.**
	G28.3 | Not used | **(TinyG) Set Absolute Axis to Defined Position**
	G29 | Go to G29 Reference Point | **(Marlin, MachineKit) Detailed Z-Probe**
	G29.1 | Not used | **(MachineKit) Set Z probe head offset**
	G29.2 | Not used | **(MachineKit) Set Z probe head offset calculated from toolhead position**
	G30 | Go To Predefined Position Through Point (G30) | **(Marlin, Smoothie, Reprap) Single Z-Probe**
	G30.1 | Set Predefined Position | 
	G31 | Straight Probe Until Skip | **(Marlin) Dock Z Probe Sled**
	G31 | Straight Probe Until Skip | **(Smoothie) Report Current Probe Status**
	G32 | Thread Cutting | **(Marlin) Undock Z Probe Sled**
	G32 | Thread Cutting | **(Smoothie) Probe Z and Calculate Z Plane**
	G33 | Spindle Synchronized Motion
	G33.1 | Rigid Tapping
	G34 | Not used
	G35 | Automatic Tool Diameter Measurement |
	G36 | Automatic Work Offset Measurement |
	G37 | Automatic Tool Length Measurement |
	G38.2 | Straight Probe To Workpiece, Report if failure |
	G38.3 | Straight Probe To Workpiece |
	G38.4 | Straight Probe Away From Workpiece, Report if failure |
	G38.5 | Straight Probe Away From Workpiece |
	G39 | Not used |
	G40 | Cancel Cutter Compensation | 
	G41 | Start Cutter Radius Compensation Left |
	G41.1 | Dynamic Cutter Compensation |
	G42 | Start Cutter Radius Compensation Right |
	G42.1 | Dynamic Cutter Compensation |
	G43 | Tool Length Offset | 
	G43 | Tool Length Compensation, Positive | 	
	G43.1 | Dynamic Tool Length Offset |
	G43.2 | Apply additional Tool Length Offset |	
	G44 | Tool Length Compensation, Negative (Fanuc, Haas) |
	G49 | Cancel Tool Length Compensation |
	G50 | Reset Scale Factors to 1.0 | 
	G51 | Set Axis Data Input Scale Factors | 
	G52 | Local Work Shift | 
	G53 | Motion In Machine Coordinate System | 
	G54 | Select Coordinate System 1 | 
	G55 | Select Coordinate System 2 | 
	G56 | Select Coordinate System 3 | 
	G57 | Select Coordinate System 4 | 
	G58 | Select Coordinate System 5 | 
	G59 | Select Coordinate System 6 | 
	G59.1 | Select Coordinate System 7 | 
	G59.2 | Select Coordinate System 8 | 
	G59.3 | Select Coordinate System 9 | 
	G60 | Unidirectional Positioning | 
	G61 | Exact Path Mode |
	G61.1 | Exact Stop Mode	|
	G62 | Automatic Corner Override | 
	G63 | Tapping Mode | 
	G64 | Continuous Mode | 
	G65 | Macro Subroutine Call | 
	G68 | Coordinate System Rotation | 
	G69 | Cancel Coordinate System Rotation |
	G70 | Bolt Hole Circle | 
	G71 | Bolt Hole Arc | 
	G72 | Bolt Holes Along and Angle | 	
	G73 | Drilling Cycle with Chip Breaking
	G74 | Reverse Tap Canned Cycle | 
	G76 | Multi-pass Threading Cycle | 
	G77 | Back Bore Canned Cycle | 
	G80 | Cancel Motion Mode | 
	G81 | Drilling Cycle |
	G82 | Drilling Cycle with Dwell |
	G83 | Drilling Cycle with Peck |
	G84 | Tapping Canned Cycle | 	
	G85 | Boring Cycle, No Dwell, Feed Out |
	G86 | Boring Cycle, Stop, Rapid Out |
	G87 | Bore/Manual Retract Canned Cycle | 
	G88 | Bore/Dwell Canned Cycle | 
	G89 | Boring Cycle, Dwell, Feed Out |
	G90 | Absolute Distance Mode |
	G09.1 | Absolute Arc Distance Mode |
	G91 | Incremental Distance Mode	| 
	G91.1 |Incremental Arc Distance Mode |
	G91.x | Reset Coordinate System Offsets |
	G92 | Set Coordinate System Offsets |
	G92.1 | Cancel Coordinate System Offsets |
	G92.2 | Cancel Offset Coordinate Systems, Do Not Reset Parameters |
	G92.3 | Apply Parameters to Offset Coordinate Systems | 
	G93 | Inverse Time Feed Rate Mode | 
	G94 | Units Per Minute Feed Rate Mode | 
	G95 | Units Per Revolution Feed Rate Mode |
	G96 | Constant Surface Speed |
	G97 | RPM Mode / Cancel Constant Surface Speed |
	G98 | Initial Level Return In Canned Cycles | 
	G99 | R-point Level Return In Canned Cycles |
	G100+ | Haas Gcodes continue from G100 to G188 |