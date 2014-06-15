##Using the Digital Digital Readout
The DIgital DIgital Read Out uses G2 on a Due to count steps an an external device and convert them back to position for display as JSON. This is useful for very fine accuracy measurement and testing.

##Building the DDRO
The digital DRO builds from the digital-dro branch of the Synthetos G2 github repo. Do this:

- clone the G2 repo
- checkout the digital-dro branch
- select the SAM3X8E as the device
- compile in Atmel Studio 6 using the the DDRO configuration (pulldown window)
- flash to a Due using the Tools/Device Programming window

You should see an OpenMoko TinyG v2 USB port when flashing is finished. Connect this to a terminal emulator such as Coolterm (our favorite).

##Hardware Setup
You will need a board-under-test and an Arduino Due (the DDRO). The board under test can be a TinyG v8 or v9, or any board that outputs individual step, direction, and ~enable signals for each axis. These should be 3.3v signals or you may blow up the Due. Up to 6 axes are supported.

Due pinouts are numbered as they appear on the board. Assumes motor mapping 1,2,3,4 to X,Y,Z,A

	Due Pin | Signal | Description
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
You should see an OpenMoko TinyG v2 USB port which will show up as usbmodem001 or something similar in the Coolterm Options / Serial Port dialog (did we mention Coolterm was our favorite?). You do not need to set a baud rate or flow control as these are handed by the USB on the Due. You may want to select terminal mode, CR line feed from the Terminal page

The following elements in the config much match the board under test.

###Motor Configuration
1 is used as an example but all motors should be set.

	Token | Element | Description / Instruction 
	------|------------|---------
	1ma | map-to-axis | Map the motors to the same axes as the board under test 
	1sa | step-angle | Same as the corresponding motor on the test board
	1tr | travel-per-revolution | Same as the corresponding motor on the test board
	1mi | microsteps | Same as the corresponding motor on the test board
	1po | polarity | Set so the steps and position go positive when the test board goes positive. Should be the same settings as on the test board, but might not be. Best to test.

###Axis Configuration
X is used as an example but all axes should be set. Only the axis mode needs to be set.

	Token | Element | Description / Instruction 
	------|------------|---------
	Xam | axis-mode | Same as the corresponding axis on the test board

###Sys Configuration

	Token | Element | Description / Instruction 
	------|------------|---------
	sv | status-verbosity | 2=VERBOSE (recommended, not mandatory)

##Use
You can use this many ways, but here's a common example

 - Set the system to JSON mode by typing { <CR>
 - Run the board under test and watch the streaming position and step count

To clear (zero) the counters type

{clc:n} or $clc

Remember to reset to JSON mode by typing a { if the latter.