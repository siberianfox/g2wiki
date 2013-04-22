This setup is designed to be as portable as possible so the project can be run from Atmel Studio 6, Apple Xcode, native Linux GCC environments, and other development environments. As a result many sources that may be normally found in the IDE environments are pulled into the project to remove as many IDE dependencies as possible. 
**NOTE: This page is not complete yet. Mostly just Notes for now**

## Getting Started
These instructions assume the following environment. Similar environments should work, but have not been tested.
* Atmel Studio 6.1-2440-beta
* Windows XP SP3 with all current patches up-to-date
* VMware 4 or 5 hosting XP on...
* Apple Macintosh OSX 10.8.3 (Mountain Lion)

### Steps for bringing up project using tinyg2.atsln or tinyg2.cppproj files from g2 github
* Buy a [SAM-ICE (Segger Jlink) and adapter cable](https://github.com/synthetos/g2/wiki/g2-in-Studio6#installing-the-sam-ice)

* Make sure your VM has at least 2.5 Gb RAM allocated to it, and your XP image (or whatever) is up to date with service packs. 

* Do a minimal install of AtmelStudio6.1-2440-beta with dotnet `AtmelStudio-6.1.2440-beta-net.exe`
 * Select the standard install locations
 * Fire it up
 * You won't need the Atmel Gallery

* Install Yagarto on Windows for Atmel Studio6.1
Ref: http://avrstudio5.wordpress.com/2013/03/07/using-winavr-with-atmel-studio-6-0-or-later/

* Get yagarto for windows. The latest version is 20121222 as of today - 4/19/13<br> http://sourceforge.net/projects/yagarto/files/YAGARTO%20for%20Windows/

* Install in windows. Let it create a yagarto-2012122 (or similar) directory in the C: drive

* Open Studio6. 
 * (1) Select `Advanced` in the `Project / TinyG2 Properties` panel. 
 * (2) Click on the `Tools --> Options --> Toolchain --> Flavour Configuration` link. 
 * (3) Select `Atmel ARM 32-Bit (CPP Language)`
 * (4) Add Flavour. Package name = `Yagarto`. Set the base to `C:\yagarto-20121222`. 
 * (5) Select it and `Set as Default`
 * (6) Set the Toolchain path to `C:\yagarto-20121222\bin` and finally hit `OK'
 * (7) Repeat steps (3) - (6) for `Atmel ARM 32-Bit (C Language)`
 * (8) Select `Toolchain Flavour - Yagarto` when you return to the main Advanced panel

### Instructions for a virgin install - i.e. no tinyg2.atsln or tinyg2.cppproj files 

* Start a `New Project`
 * Select `C++ Executable` project
 * Name `TinyG2`
 * Directory - browse to main project directory. Be aware that AS6 doesn;t know what you called your VMware attached drive so you have to find in `My Network Places \ Shared Folders on vm-ware`. How very Redmond.
 * Solution Name `g2`
 * Check `Create Directory for Project`. If you want to change the directory structure you can move things around and go edit the tinyg.atsln file so it can find the project sub-directory (relative path name).
 * Select the chip you want. THe Arduino Due is a ATSAM3X8E

* Populate the project directories. At this point you have a choice. Visual Studio does nto have a way to import an entire existing directory in a single operation. So you can laboriously add every directory, move the files in, then select the file as existing items ... all the way down the hierarchy, or you can edit the cppproj file XML directly.

* Port in the entire CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`

## Useful Facts about Studio 6.0

#### Renaming A Project
You can rename a project by:
* deleting the .atsuo file
* changing the name of the OLD.atsln and OLD.cppproj files
* editing the NEW.atsln file with a text editor (not VisualStudio) to agree with the new name
* I did not change the GUIDs but this seems not to have mattered

## Notes for Rob Regarding new g2 Project Structure
_Some of this is already outdated_
* Started with the giseburt/g2 fork from 3/29/13
* Replaced the contents of the libsam directory with the libsam from Arduino 1.5.x-cores branch - these dirs:
<pre>
Arduino/hardware/arduino/sam/system/libsam/include
Arduino/hardware/arduino/sam/system/libsam/source
</pre>
* Added the following lines to chip.h in the libsam dir
<pre>
 /**** All fixes needed for this to compile are concentrated here ***/
 /* The following modifies the SAM3XA_SERIES #define from the sam.h file to fix compilation problems in adc.h
   Ref: http://asf.atmel.no/docs/latest/common.services.calendar.example2.stk600-rcuc3d/html/group__sam__part__macros__group.html
 */
 //#define SAM3XA_SERIES (SAM3A4 || SAM3A8)	// original define in sam.h file
 #undef SAM3XA_SERIES
 #define SAM3XA_SERIES (SAM3X4 || SAM3X8 || SAM3A4 || SAM3A8)

 #define USB_PID USB_PID_DUE
/**** TO HERE ****/
</pre>
* Added back these Studio6 auto-generated dirs:
<pre>
CMSIS/linker_scripts
CMSIS/src
</pre>

* Added Rob's Makefile and set the TinyG2 project to use it. But here's what I got:
<pre>
------ Rebuild All started: Project: TinyG2, Configuration: Debug ARM ------
Build started.
Project "TinyG2.cppproj" (ReBuild target(s)):
Target "PreBuildEvent" skipped, due to false condition; ('$(PreBuildEvent)'!='') was evaluated as (''!='').
Target "CoreRebuild" in file "C:\Program Files\Atmel\Atmel Studio 6.0\Vs\Compiler.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG2\TinyG2.cppproj" (target "ReBuild" depends on it):
	Task "RunCompilerTask"
		C:\Program Files\Atmel\Atmel Studio 6.0\make\make.exe -C "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG2" -f "Makefile" clean all 
		make: Entering directory `Z:/Alden/Projects/proj64_ArduinoDue/g2/TinyG2'
		rm -fR build/SAM3X8E bin/SAM3X8E
		make: Leaving directory `Z:/Alden/Projects/proj64_ArduinoDue/g2/TinyG2'
		make: *** No rule to make target `bin/SAM3X8E', needed by `all'.  Stop.
	Done executing task "RunCompilerTask" -- FAILED.
Done building target "CoreRebuild" in project "TinyG2.cppproj" -- FAILED.
Done building project "TinyG2.cppproj" -- FAILED.
 Build FAILED.
 ========== Rebuild All: 0 succeeded, 1 failed, 0 skipped ==========
</pre>