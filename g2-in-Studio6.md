This setup is designed to be as portable as possible so the project can be run from Atmel Studio 6, Apple Xcode, native Linux GCC environments, and other development environments. As a result many sources that may be normally found in the IDE environments are pulled into the project to remove as many IDE dependencies as possible. 

The project setup uses Studio6 as an IDE and debugging platform, but does not use the Studio6 toolchain, auto-generated makefile, libraries or sources. It uses the Yagarto ARM GCC toolchain to get a newlib that's compiled for printf() floatinf point support. It also uses a Makefile provided by the project. This is an attempt to keep the project as portable as possible to non-Studio6 build environments

These instructions assume the following environment. Similar environments should work, but have not been tested.
* Atmel Studio 6.1-2440-beta
* Windows XP SP3 with all current patches up-to-date
* VMware 4 or 5 hosting XP on...
* Apple Macintosh OSX 10.8.3 (Mountain Lion)
* Yagarto-20121222

## Setting Up Studio6 When You Have Project Files From the g2 Github
* For starters make sure your VM has at least 2.5 Gb RAM allocated to it, and your XP image (or whatever) is up to date with service packs. 
* Clone the [g2 Github repository](https://github.com/synthetos/g2)
* Get the following:
 * [Atmel Studio 6.1-2440-beta](http://www.atmel.com/tools/atmelstudio.aspx). Get the one with .NET you don't have a current .NET install on your system
 * [Atmel SAM-ICE (Segger Jlink)](http://www.mouser.com/ProductDetail/Atmel/AT91SAM-ICE/?qs=%2fha2pyFadujAZ79HQyfG%252bJm4Wmz2%2fLVln%2foieqku2gI%3d) 
 * [Olimex JTAG adapter](http://www.mouser.com/ProductDetail/Olimex-Ltd/ARM-JTAG-20-10/?qs=sGAEpiMZZMt%2f9hUFx8MktsRg8ShTvwMQusYCyASUbpU%3d)
* Install `AtmelStudio-6.1.2440-beta-net.exe` or `AtmelStudio-6.1.2440-beta.exe`
 * Select the standard install locations
 * When asked, you won't need the Atmel Gallery
* Install Yagarto on Windows
 * Download [Yagarto GNU ARM Toolchain for windows](http://www.yagarto.de/#download). The latest version is 20121222 as of today - 4/19/13
 * Install in windows. Install everything. Let it create a yagarto-2012122 (or similar) directory in the C: drive
* Bootstrap Yagarto into the project.
 * (0) Don't attempt to open TinyG2.atsln or TinyG2.cppproj yet. They will crash because Yagarto is not yet installed
 * (1) Open Studio6 by clicking on the desktop icon
 * (2) Select `New Project` and make it a `GCC C++ Executable Project`
 * (3) Just accept all the default names and file locations - it won't matter
 * (4) Select the device to be `ATSAM3X8E`
 * (5) Once the project opens select `Project / GccApplication1 Properties`. Go to the `Advanced` panel
 * (6) Click on the `Tools --> Options --> Toolchain --> Flavour Configuration` link. 
 * (7) Select `Atmel ARM 32-Bit (CPP Language)`
 * (8) Add Flavour. Package name = `Yagarto`. Set the base to `C:\yagarto-20121222`. Click `Add`
 * (9) Once you are back in the `Package Configuration` dialog select `Yagarto` and `Set as Default`
 * (10) Set the Toolchain path to `C:\yagarto-20121222\bin` and finally hit `OK`
 * (11) Repeat steps (6) - (10) for `Atmel ARM 32-Bit (C Language)`
 * (12) When you return to the Advanced panel select `Toolchain Flavour - Yagarto`
 * (13) Exit to project and say YES to save.
* If you now click on the file `tinyg2.atsln` it should open the project and you should be able to compile and debug
 * NB: At the current time the REBUILD does not work. It will clean everything and then fail. Use BUILD only. If you need to rebuild realize that it's a 2 step process. Rebuild (with the error), then build.
* If you navigate to the Reference directory and run `make` (no arguments) from the command line it will pull in the Newlib sources and unzip them. This can be handy for providing source for symbolic debugging if you want to dive down that low.

## Setting Up Studio6 When You Don't Have Project Files From the g2 Github (Virgin Install)
These instructions assume that you have all the sources, including these directories and their sub-directories. You just don't have the project files, or you need to regenerate them for some reason. _Note: These instructions are incomplete._
* First, do everything from above. Get AS6 and set up Yagarto
* Make sure you have these source directories:
 * arduino
 * CMSIS
 * motate
 * platform
 * settings
 * variants
 * (bin, build and other dirs will be added automatically) 
* Start a `New Project`
 * Select `GCC C++ Executable` project
 * Name is `TinyG2`
 * Directory - browse to main project directory. Be aware that VisualStudio doesn't know what you called your VMware attached drive so you have to find in `My Network Places \ Shared Folders on vm-ware`. Or type it in directly.
 * Solution Name `g2`
 * Check `Create Directory for Project`. If you want to change the directory structure you can move things around and go edit the tinyg.atsln file so it can find the project sub-directory (relative path name).
 * Select the chip you want. The Arduino Due is a ATSAM3X8E in the SAM3 series
* Populate the project directories. Since we are using an external makefile it's not necessary to include all sources, but if you want to edit things in AS6 it's handy to have the files in the project viewer.  
 * At this point you have a choice. Visual Studio does not have a way to import an entire existing directory in a single operation. So you can laboriously add every directory, move the files in, then select the file as existing items ... all the way down the hierarchy, or you can edit the cppproj file XML directly.
* Once you have added whatever files you want select `External Makefile` in the project properties `Build` panel
 * Make file name is `Makefile`
 * Build command line is `TinyG2.elf COLOR=0 VERBOSE=1`
 * Clean commandline is `clean`
* Be sure to exit and save the project to save your work.

* Port in the entire CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`

## Useful Facts about Studio 6.0

#### Renaming A Project
You can rename a project by:
* deleting the .atsuo file
* changing the name of the OLD.atsln and OLD.cppproj files
* editing the NEW.atsln file with a text editor (not VisualStudio) to agree with the new name
* I did not change the GUIDs but this seems not to have mattered
