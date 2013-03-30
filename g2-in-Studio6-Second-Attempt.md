The [first attempt](https://github.com/synthetos/g2/wiki/g2-in-Studio6) worked OK, but we wanted to get all the sources into the project and not rely on the Studio6 CMSIS hierarchy and other baked-in assumptions. This is so the project can be run from Apple Xcode, native Linux GCC environments, and other non-Studio6 development environments.

**Just Notes for now**

* Buy a [SAM-ICE (Segger Jlink) and adapter cable](https://github.com/synthetos/g2/wiki/g2-in-Studio6#installing-the-sam-ice)

* Do a minimal install of AtmelStudio6.0 build 1996 dotnet `as6installer-6.0.1996-net.exe`
 * Select the standard install locations
 * Fire it up
 * You don't need the Atmel Gallery

* Port in the entiure CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`