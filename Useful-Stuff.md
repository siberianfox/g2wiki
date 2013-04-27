##Things We've Learned - Mostly The Hard Way
Most of these assume the following development setup: Atmel Studio 6.1-2440-beta on Windows XP in VMWare (version 4 or 5) running on Mac OSX 10.8.3 (Mountain Lion) and using Coolterm 1.4 on the Mac for the USB terminal window. Using the SAM-ICE for programming and real-time debugging from Studio 6. We also use Apple Xcode for editing and compilation and then use Studio6 for programming and debugging.

In no particular order:

###General
* If USB stops working or works erratically unplug and replug the Arduino and the SAM-ICE
* If the Arduino Due indicates that it is connecting to your VM as "Atmel Mode" instead of "Arduino Due", it's because its in the ARM bootloader. Unplug and reconnect the USB and it should connect as a Due. This is apparent when you have a Windows (or other) VM running in Mac and you get the mac dialog box. In other configs the USB might just stop working with no indication. Plug and replug.

### Studio6 related
* If you install Studio6.x it may create a /Debug directory. Delete this directory. It will confuse the debugger into looking there for the tinyg2.elf file, which you don;t want. You want the AS6 debugger to look for tinyg2.elf in the TinyG2 directory - which it will if you delete the Debug dir.
* When you plug the SAM-ICE in it may be recognized in SWD or JTAG mode. You want it in JTAG mode or the cycle counter profiling won't work.
* If you start debug and are presented with a dialog box to select the SAM-ICE - select it, then start debug again. For some reason (timeout?) that first session does not connect to the ICE.
* The Makefile currently can't do a Rebuild Solution or Rebuild Project. It will do the clean then fail in the build. If you need to do a complete rebuild run Clean first, then do a Build. 

### VMware related
* You need to go into the VMware USB Advanced dialog to make sure the Arduino Due always connects to the mac. Don't check the blue box - it means the USB is connected to the VM. Confusing.* Atmel Studio 6.x still doesn't display ASCII in arrays. We included an ASCII table at the end of xio.h so you can look up what the array says. Yuk.

