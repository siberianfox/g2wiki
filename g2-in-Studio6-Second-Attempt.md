The [first attempt](https://github.com/synthetos/g2/wiki/g2-in-Studio6) worked OK, but we wanted to get all the sources into the project and not rely on the Studio6 CMSIS hierarchy and other baked-in assumptions. This is so the project can be run from Apple Xcode, native Linux GCC environments, and other non-Studio6 development environments.

**Just Notes for now**

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

* Port in the entire CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`