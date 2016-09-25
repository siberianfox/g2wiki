See also [Gcodes](Gcodes)

##Mcode Summary 

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