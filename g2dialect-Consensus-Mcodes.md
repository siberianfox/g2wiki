See also [Consensus Gcode](g2dialect-Consensus-Gcode)

##Consensus Mcode Usage
This table lists rough consensus usage from the above sources.

	Gcode | Command | Usage / Notes
	--------|-------------|-----------------------------
	M0 | Program Stop | Stop motion
	M1 | Optional Program Stop |
	M2 | Program End |
	M3 | Spindle On, Clockwise |
	M4 | Spindle On, Counterclockwise | 
	M5 | Spindle Stop |
	M6 | Tool Change | 
	M7 | Mist Coolant On | 
	M8 | Flood Coolant On | 
	M9 | Mist And Flood Coolant Off | 
	M30 | Program End | 
	M48 | Enable Feed & Spindle Overrides | cnc
	M49 | Disable Feed & Spindle Overrides | cnc
	M50 | Feed Override Control | cnc
	M51 | Spindle Override Control | cnc
	M60 | Stop | cnc
	M62 | Digital Output - Turn on sync w/motion | cnc
	M63 | Digital Output - Turn off sync w/motion | cnc
	M64 | Digital Output - Turn on immediately | cnc
	M65 | Digital Output - Turn off immediately | cnc
	M66 | Wait on Input | cnc
	M67 | Analog Output Control | cnc
	M68 | Analog Output Control | cnc
