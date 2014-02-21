This page documents the current state of the g2 build and test environments. 

We are currently using both an Xcode (mac) environment and an Atmel Studio 6.1 (win) environment. Typically we run both on a mac with the AS6 in a VM. They each do certain things better. 
* Xcode is more familiar for Mac users and complies faster than Studio6 if AS6 is running on a virtual machine.
* AS6 has a really nice realtime debug environment using the SAM-ICE that displays registers, cycle counters and profiling, memory structures, IO devices, call stack, etc. 

## Setting Up Atmel Studio 6
These instructions walk through setting up a Studio 6 environment for use with existing project files available from the g2 github. (We will also post virgin setup instructions later).

See [g2 in Studio 6](https://github.com/synthetos/g2/wiki/g2-in-Studio6)

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

(Estee: `arm-non-eabi-gcc` is not found in the yagarto-4.7.2 directory after it has been installed)

* Now move the yagarto-4.7.2 directory to /usr/local. OSX hides this directory from you. Type <cmd><shift>g /usr/local to open it. from the "Go to the folder:" dialog box.

* Drag the yagarto-4.7.2 directory (not the app) into /usr/local. You will probably need to enter these commands first:
 * `sudo -s`  (followed by putting in your password)
 * `echo '/usr/local/yagarto-4.7.2/bin' >> /etc/paths.d/10-yagarto`
 * Now you should be able to drag it in

close the terminal window, open a new one 
sudo -a (password entry)
echo '/usr/local/yagarto-4.7.2/bin' >> /etc/paths.d/10-yagarto

* It’s in $(ROOT)/Reference, and I .gitignore’d it. There’s a Makefile in there to grab and decompress the newlib source. We can easily add other sources as needed.
