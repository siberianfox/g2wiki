This page documents the current state of the g2 build and test environments. 

We are currently using both an Xcode (mac) environment and an Atmel Studio 6.1 (win) environment. Typically we run both on a mac with the AS6 in a VM. They each do certain things better. 
* The big plus for Xcode is usability and speed. 
* AS6 has a really nice realtime debug environment using the SAM-ICE that displays registers, cycle counters and profiling, memory structures, IO devices, call stack, etc. 

##Still To Go
* Get Rebuild all to work - i.e. `make clean all`
* Re-point the build size dialog to the root directory so it works again

##Getting Floating Point printf() to Work
The newlib distributed with Studio6 does not have floating point printf() enabled. You need to get a newlib that does and point the project to it. We used the Yagarto toolchain. These instructions explain how to use the Yagarto toolchain as the default toolchain for Mac (Xcode), and Windows (Studio6)

### Yagarto on the Mac for Xcode
These instructions apply to Max OSX using Xcode 4.4 or later.
* If you don't already have them, install Xcode Command Line tools from Xcode / Preferences / Downloads
* Get yagarto (4.7.2): http://sourceforge.net/projects/yagarto/files/YAGARTO%20for%20Mac%20OS%20X/20121222/ Just the top file, the DMG. (It doesn’t mess with anything. Better, it’s installing a generic ARM toolchain.)
* Open and install yagarto. Unpack the yagarto .dmg. You should see the yagarto-4.7.2.app (or similar) in a new finder window. Move this to your home directory (~). Run the app. 
 * If you can't run it by double clicking, right click it and hit "Open". You should see a yagarto-4.7.2 directory in your home dir when it's done.
 * In Mountain Lion the yagarto-4.7.2 dir may not show up in the Finder window. You should be able to see it from the command line in a terminal window. (`cd ~`  `ls`) In this case enter `open yagarto-4.7.2` from the command line. It should show up in the Finder.
* Close your terminal window and open a new one. Enter these commands from the command line
 * `sudo -s`
 * `arm-non-eabi-gcc`

* Now move the yagarto-4.7.2 directory to /usr/local. OSX hides this directory from you. Type <cmd><shift>g /usr/local to open it. from the "Go to the folder:" dialog box.

* Drag the yagarto-4.7.2 directory (not the app) into /usr/local. You will probably need to enter these commands first:
 * `sudo -s`  (followed by putting in your password)
 * `echo '/usr/local/yagarto-4.7.2/bin' >> /etc/paths.d/10-yagarto`
 * Now you should be able to drag it in

close the terminal window, open a new one 
sudo -a (password entry)
echo '/usr/local/yagarto-4.7.2/bin' >> /etc/paths.d/10-yagarto

* It’s in $(ROOT)/Reference, and I .gitignore’d it. There’s a Makefile in there to grab and decompress the newlib source. We can easily add other sources as needed.

### Yagarto on Windows for Atmel Studio6.1
These instructions apply to Atmel Studio 6.1. We are using a Windows XP VMware virtual machine running on OSX 10.8.3 (Mountain Lion), but other OS's should be similar. <br>
Ref: http://avrstudio5.wordpress.com/2013/03/07/using-winavr-with-atmel-studio-6-0-or-later/

* Get yagarto for windows. The latest version is 20121222 as of today - 4/19/13<br> http://sourceforge.net/projects/yagarto/files/YAGARTO%20for%20Windows/

* Install in windows. Let it create a yagarto-2012122 (or similar) directory in the C: drive

* Open Studio6. 
 * Select `Advanced` in the `Project / TinyG2 Properties` panel. 
 * Click on the `Tools --> Options --> Toolchain --> Flavour Configuration` link. 
 * Select `Atmel ARM 32-Bit (CPP Language)`
 * Add Flavour. Package name = `Yagarto`. Set the base to `C:\yagarto-20121222`. 
 * Select it and `Set as Default`
 * Do the same for `Atmel ARM 32-Bit (C Language)`
 * Set the Toolchain path to `C:\yagarto-20121222arm-none-eabi\bin` and finally hit `OK'
 * Select `Toolchain Flavour - Yagarto` when you return to the main Advanced panel

##Things We've Learned - Mostly The Hard Way
In no particular order:
* When you plug the SAM-ICE in it may be recognized in SWD or JTAG mode. You want it in JTAG mode or the cycle counter profiling won;t work
* If the Arduino Due indicates that it is connecting to your VM as "Atmel Mode" instead of "Arduino Due", it's because its in the ARM bootloader. Unplug and reconnect the USB and it should connect as a Due. This is apparent when you have a Windows (or other) VM running in Mac and you get the mac dialog box. In other configs the USB might just stop working with no indication. Plug and replug.
* Atmel Studio 6.x still doesn't display ASCII in arrays. We included an ASCII table at the end of xio.h so you can look up what the array says. Yuk.
* If you install Studio6.x it may create a /Debug directory. Delete this directory. It will confuse the debugger into looking there for the tinyg2.elf file, which you don;t want. You want the AS6 debugger to look for tinyg2.elf in the TinyG2 directory - which it will if you delete the Debug dir.