Please note: We do not yet have instructions for Windows or Linux, but the steps are similar.

PROGRAMMING THE DUE FROM OS X
=============================

###Step 1 - Get the TinyG2.elf file
Get the tinyg2.elf file from the master branch:
https://github.com/synthetos/g2/blob/master/TinyG2/TinyG2.elf

Click on the RAW tab and it should download it for you. Go to Step 2.

Alternately, you can get the tinyg2.hex file here:
https://raw.github.com/synthetos/g2/master/TinyG2/TinyG2.hex

You need the RAW page, that looks like this:
<pre>
:0200000280007C
:1000000000800820C1B1080035B2080035B20800F0
:1000100035B2080035B2080035B208000000000013
...
</pre>

...not the HTML formatted page that just shows the hex in a window. Right click to "Save As" tinyg2.hex (you may need to get rid of a .txt extension during this save). Save it in some known directory that you can get back to. The downloaded file should be about 350K bytes. If it's much bigger than that you got the HTML file by mistake.

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