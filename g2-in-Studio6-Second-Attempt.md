The [first attempt](https://github.com/synthetos/g2/wiki/g2-in-Studio6) worked OK, but we wanted to get all the sources into the project and not rely on the Studio6 CMSIS hierarchy and other baked-in assumptions. This is so the project can be run from Apple Xcode, native Linux GCC environments, and other non-Studio6 development environments.

**Just Notes for now**

## Getting Started
* Buy a [SAM-ICE (Segger Jlink) and adapter cable](https://github.com/synthetos/g2/wiki/g2-in-Studio6#installing-the-sam-ice)

* Make sure your VM has at least 2.5 Gb RAM allocated to it, and your XP image (or whatever) is up to date with service packs. 

* Do a minimal install of AtmelStudio6.0 build 1996 dotnet `as6installer-6.0.1996-net.exe`
 * Select the standard install locations
 * Fire it up
 * You don't need the Atmel Gallery

* Start a `New Project`
 * Select `C++ Executable` project
 * Name `TinyG2`
 * Directory - browse to main project directory. Be aware that AS6 doesn;t know what you called your VMware attached drive so you have to find in `My Network Places \ Shared Folders on vm-ware`. How very Redmond.
 * Solution Name `g2`
 * Check `Create Directory for Project`. If you want to change the directory structure you can move things around and go edit the tinyg.atsln file so it can find the project sub-directory (relative path name).
 * Select the chip you want. THe Arduino Due is a ATSAM3X8E

* Populate the project directories. At this point you have a choice. Visual Studio does nto have a way to import an entire existing directory in a single operation. So you can laboriously add every directory, move the files in, then select the file as existing items ... all the way down the hierarchy, or you can edit the cppproj file XML directly.

* Port in the entire CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`

## Notes for Rob Regarding new g2 Project Structure
* Started with the giseburt/g2 fork from 3/29/13
* Replaced the contents of the libsam directory with the libsam from Arduino 1.5.x-cores branch - these dirs:
<pre>
Arduino/hardware/arduino/sam/system/libsam/include
Arduino/hardware/arduino/sam/system/libsam/source
</pre>
* Added the following lines to chip.h in the libsam dir
<pre>

</pre>
* Added back these Studio6 auto-generated dirs:
<pre>
CMSIS/linker_scripts
CMSIS/src
</pre>
