This page documents the current state of the g2 build and test environments. We are currently using both an Xcode (mac) environment and an Atmel Studio 6.1 (win) environment. Typically we run both on a mac with the AS6 in a VM. THey both do certain things better. The big win on Xcode is usability and speed. AS6 has a really nice realtime debug environment using the SAM-ICE that displays registers, cycle counters and profiling, memory structures, IO devices, etc. 

##Things We've Learned - Mostly The Hard Way
In no particular order:
* When you plug the SAM-ICE in it may be recognized in SWD or JTAG mode. You want it in JTAG mode or the cycle counter profiling won;t work
* If the Arduino Due indicates that it is connecting to your VM as "Atmel Mode" instead of "Arduino Due", it's because its in the ARM bootloader. Unplug and reconnect the USB and it should connect as a Due. This is apparent when you have a Windows (or other) VM running in Mac and you get the mac dialog box. In other configs the USB might just stop working with no indication. Plug and replug.
* Atmel Studio 6.x still doesn't display 