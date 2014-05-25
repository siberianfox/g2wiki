##Using the Digital Digital Readout

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

The following elements in the config much match the board under test:

###Motor Configuration

	Token | Element | Description / Instruction 
	------|------------|---------
	ma | GROUND | 
	2 | X step | 


