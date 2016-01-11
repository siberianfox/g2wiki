_This page is for uploading an already compiled G2 binary to a target board without a debugger from OS X. Please see [Getting Started with G2](Getting-Started-with-G2) for information about other options._

Note that these instructions are for flashing TinyG2 to a Arduino Due or TinyG v9, and even though it *requires* the Arduino IDE it doesn't use the Arduino IDE. You will still need to use the command line.

This is also for flashing *without* a hardware debugger such as the Atmel ICE, Atmel SAM-ICE, or Segger JLink.

Please see [this guide](Debugging-G2-on-OSX-with-GDB-and-Atmel-ICE#flashing-the-firmware-onto-the-board) for instructions on flashing *with* a hardware debugger

###Step 1 - Get the TinyG2.elf file

**Option 1** - Compile your own using the instructions in [[Compiling G2 on OS X (with Xcode)]]

**Option 2** - Get the tinyg2.bin binary firmware files from http://synthetos.github.io/g2/
###Step 2 - Install the Arduino Due environment

_If you already have the Arduino Due environment installed you can skip this step_

Download the Arduino 1.6.x from http://arduino.cc/en/Main/Software for OS X.

Expand the downloaded zip file and move Arduino.app to /Applications on your hard drive. (Often, this is found on the sidebar of Finder windows.)

###Step 3 - Clone the g2 git repository to your local disk

You need some of the scripts in the https://github.com/synthetos/g2 git repository, so clone it to your local disk.

###Step 4 - Program TinyG2 onto the Due

Open Terminal.app, then change to the Resources/TinyG2-OSX-Programmer/ subdirectory in the g2 repository you just cloned.  One way to do that is:

1. Type "cd " (That's lowercase "c", then "d", then space), DON'T press return yet.
2. Then drag the Resources/TinyG2-OSX-Programmer/ *folder* from the cloned g2 repository to that Terminal.app window. It should type in a path for you.
3. Now press return.

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