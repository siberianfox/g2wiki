The [first attempt]() worked OK, but we wanted to get all the sources into the project and not rely on the Studio6 CMSIS hierarchy and other baked-in assumptions. This is so the project can be run from Apple Xcode, native Linux GCC environments, and other non-Studio6 development environments.

**Just Notes for now**

* Port in the entiure CMSIS_Atmel hierarchy - change `CMSIS_Atmel` to `CMSIS`
