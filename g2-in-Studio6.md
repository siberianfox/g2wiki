This setup is designed to be as portable as possible so the project can be run from Atmel Studio 6, Apple Xcode, native Linux GCC environments, and other development environments. As a result many sources that may be normally found in the IDE environments are pulled into the project to remove as many IDE dependencies as possible. 
**NOTE: This page is not complete yet. Mostly just Notes for now**

## Getting Started
These instructions assume the following environment. Similar environments should work, but have not been tested.
* Atmel Studio 6.1-2440-beta
* Windows XP SP3 with all current patches up-to-date
* VMware 4 or 5 hosting XP on...
* Apple Macintosh OSX 10.8.3 (Mountain Lion)

### Steps for bringing up project using the Studio6 project files from the g2 github
* Clone the [g2 Github repository](https://github.com/synthetos/g2)
* Make sure your VM has at least 2.5 Gb RAM allocated to it, and your XP image (or whatever) is up to date with service packs. 
* Get the following:
 * [Atmel Studio 6.1-2440-beta](http://www.atmel.com/tools/atmelstudio.aspx). Get the one with .NET you don't have a current .NET install on your system
 * [Atmel SAM-ICE (Segger Jlink)](http://www.mouser.com/ProductDetail/Atmel/AT91SAM-ICE/?qs=%2fha2pyFadujAZ79HQyfG%252bJm4Wmz2%2fLVln%2foieqku2gI%3d) 
 * [Olimex JTAG adapter](http://www.mouser.com/ProductDetail/Olimex-Ltd/ARM-JTAG-20-10/?qs=sGAEpiMZZMt%2f9hUFx8MktsRg8ShTvwMQusYCyASUbpU%3d)
* Install `AtmelStudio-6.1.2440-beta-net.exe` or `AtmelStudio-6.1.2440-beta.exe`
 * Select the standard install locations
 * You won't need the Atmel Gallery
* Install Yagarto on Windows
 * Download [Yagarto for windows](http://www.yagarto.de/#download). The latest version is 20121222 as of today - 4/19/13
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
* If you click on the file `tinyg2.atsln` if should open the project and you should be able to compile and debug

### Instructions for a virgin install - i.e. no tinyg2.atsln or tinyg2.cppproj files 
(These instructions are incomplete)
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
