##Things We've Learned - Mostly The Hard Way
Most of these assume the following development setup: Atmel Studio 6.1 on Windows XP in VMWare 5 running on Mac OSX 10.8.3 (Mountain Lion) and using Coolterm 1.4 on the Mac for the USB terminal window. 

In no particular order:

###General
* If USB stops working or works erratically unplug and replug the Arduino and the SAM-ICE
* If the Arduino Due indicates that it is connecting to your VM as "Atmel Mode" instead of "Arduino Due", it's because its in the ARM bootloader. Unplug and reconnect the USB and it should connect as a Due. This is apparent when you have a Windows (or other) VM running in Mac and you get the mac dialog box. In other configs the USB might just stop working with no indication. Plug and replug.

### Studio6 related
* If you install Studio6.x it may create a /Debug directory. Delete this directory. It will confuse the debugger into looking there for the tinyg2.elf file, which you don;t want. You want the AS6 debugger to look for tinyg2.elf in the TinyG2 directory - which it will if you delete the Debug dir.
* When you plug the SAM-ICE in it may be recognized in SWD or JTAG mode. You want it in JTAG mode or the cycle counter profiling won't work

### VMware related
* You need to go into the VMware USB Advanced dialog to make sure the Arduino Due always connects to the mac. Don't check the blue box - it means the USB is connected to the VM. Confusing.* Atmel Studio 6.x still doesn't display ASCII in arrays. We included an ASCII table at the end of xio.h so you can look up what the array says. Yuk.