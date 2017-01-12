This page lists common Mcodes used across NIST, Fanuc, Haas, Tormach, LinuxCNC and are typically documented in the CNC Cookbook and other sources. It does not list the numerous Mcodes used in the 3D printing arena, as these are not used or may be in conflict with standard CNC machines.

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
	M19 | Orient Spindle | (Fanuc, Haas)
	M30 | Program End | 
	M48 | Enable Feed & Spindle Overrides |
	M49 | Disable Feed & Spindle Overrides |
	M50 | Feed Override Control |
	M51 | Spindle Override Control |
	M60 | Pallet Change Pause | (LinuxCNC)
	M62 | Digital Output - Turn on sync w/motion | (LinuxCNC)
	M63 | Digital Output - Turn off sync w/motion | (LinuxCNC)
	M64 | Digital Output - Turn on immediately | (LinuxCNC)
	M65 | Digital Output - Turn off immediately | (LinuxCNC)
	M66 | Wait on Input | (LinuxCNC)
	M67 | Analog Output Control | (LinuxCNC)
	M68 | Analog Output Control | (LinuxCNC)
	M98 | Call Macro/Subprogram |
	M99 | Return from Macro/Subprogram |

**Mcodes 100 and above are user defined**
