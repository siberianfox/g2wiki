_This page is for uploading an already compiled G2 binary to a target board without a debugger from OS X. Please see [Getting Started with G2](Getting-Started-with-G2) for information about other options._

###Step 1 - Get the TinyG2.elf file

**Option 1** - Compile your own using the instructions in [[Compiling G2 on OS X (with Xcode)]]

**Option 2** - Get the tinyg2.bin binary firmware files from http://synthetos.github.io/g2/
###Step 2 - Install the Arduino Due environment

_If you already have the Arduino Due environment installed you can skip this step_

Download the Arduino 1.5 BETA (not the 1.0.5 release at the top of the page) from http://arduino.cc/en/Main/Software for OS X.

Expand the downloaded zip file and move Arduino.app to /Applications on your hard drive. (Often, this is found on the sidebar of Finder windows.)

###Step 3 - Program TinyG2 onto the Due

Open Terminal.app and change to the directory with this programmer:

* Type "cd " (That's lowercase "c", then "d", then space), DON'T hit return.
* Now drag the *folder* that you found this ReadMe.txt file in to the Terminal.app window that has the "cd " you just typed. It should type in a path for you.
* Now hit return.

Either place the .elf file you want to program into that directory or remove the trailing version number from the elf file that is present; e.g. 

	mv TinyG2.elf.13.01 TinyG2.elf

Plug a USB cable from the computer to the Programming port of the Due (the one closest to the power jack). Make sure there are no shields, programmers, debuggers or other devices plugged into the Due. The Due does not need external power for programming - it will be powered by the USB programming cable.

Copy and paste the following line into the Terminal window:

	./DueFromOSX.sh -f TinyG2.elf -p /dev/cu.usbmodem*

The output should look something like this (the numbers after cu.usbmodem will be different):

	Forcing reset using 1200bps open/close on port /dev/cu.usbmodem241311
	wait...
	Starting programming of file TinyG2.elf -> TinyG2.bin on port cu.usbmodem241311
	Erase flash
	Write 119380 bytes to flash
	[==============================] 100% (467/467 pages)
	Verify 119380 bytes of flash
	[==============================] 100% (467/467 pages)
	Verify successful
	Set boot flash true
	CPU reset.

Now disconnect the Due and plug the USB cable into the other port. You now have a Due/TinyG2.