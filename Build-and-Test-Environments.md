This page documents the current state of the g2 build and test environments. 

We are currently using both an Xcode (mac) environment and an Atmel Studio 6.1 (win) environment. Typically we run both on a mac with the AS6 in a VM. They each do certain things better. 
* The big plus for Xcode is usability and speed. 
* AS6 has a really nice realtime debug environment using the SAM-ICE that displays registers, cycle counters and profiling, memory structures, IO devices, call stack, etc. 

##Getting Floating point printf() to work and other newlib issues
The newlib distributed with AS6 does not have floating point printf enabled. You need to get a newlib that does and point the project to it. These instructions explain how to do this on Max OSX using Xcode 4.4 or later.

* If you don't already have them, install Xcode Command Line tools from Xcode / Preferences / Downloads

* DL and install: http://sourceforge.net/projects/yagarto/files/YAGARTO%20for%20Mac%20OS%20X/20121222/ (Just the top file, the DMG.) It doesn’t mess with anything. Better, it’s installing a generic ARM toolchain.

* Make 

* It’s in $(ROOT)/Reference, and I .gitignore’d it. There’s a Makefile in there to grab and decompress the newlib source. We can easily add other sources as needed.



##Things We've Learned - Mostly The Hard Way
In no particular order:
* When you plug the SAM-ICE in it may be recognized in SWD or JTAG mode. You want it in JTAG mode or the cycle counter profiling won;t work
* If the Arduino Due indicates that it is connecting to your VM as "Atmel Mode" instead of "Arduino Due", it's because its in the ARM bootloader. Unplug and reconnect the USB and it should connect as a Due. This is apparent when you have a Windows (or other) VM running in Mac and you get the mac dialog box. In other configs the USB might just stop working with no indication. Plug and replug.
* Atmel Studio 6.x still doesn't display ASCII in arrays. We included an ASCII table at the end of xio.h so you can look up what the array says. Yuk.
* If you install Studio6.x it may create a /Debug directory. Delete this directory. It will confuse the debugger into looking there for the tinyg2.elf file, which you don;t want. You want the AS6 debugger to look for tinyg2.elf in the TinyG2 directory - which it will if you delete the Debug dir.
