##Documentation for the digital digital readout.

##Building the DDRO
The Digital DRO builds from the digital-dro branch of the Synthetos G2 github repo. Do this:

- Get the digital-dro branch
- Compile the DDRO configuration
- Flash to a Due

You should see an OpenMoko TinyG v2 USB port when flashing is finished. Connect this to a terminal emulator such as Coolterm (our favorite).

##Hardware Setup
You will need a board-under-test and an Arduino Due (the DDRO). The board under test can be a TInyG v8 or v9, or any board that outputs individual step, direction, and enables for each axis. Up to 6 axes are supported.

Due pinouts are numbered as they appear on the board. Assumes motor mapping 1,2,3,4 to X,Y,Z,A

	Pin | Signal | Description
	------|------------|---------
	14 | GROUND | 
	2 | X step | 
	5 | X dir | 
	22 | X enable | 
	3 | Y step | 
	6 | Y dir | 
	25 | Y enable | 
	4 | Z step | 
	7 | Z dir | 
	28 | Z enable | 
	31 | A step | 
	32 | A dir | 
	33 | A enable | 

#Software Setup
You should see an OpenMoko TinyG v2 USB port when flashing is finished. Connect this to a terminal emulator such as Coolterm (our favorite). Connect Coolterm to the usbmodem001 or some similar identifier. You do not need to set a baud rate or flow control. You may want to select terminal mode, CR line feed.

The following elements in the config much match the board under test:

###Motor Configuration

	Token | Element | Description / Instruction 
	------|------------|---------
	ma | GROUND | 
	2 | X step | 


