# Programming the TinyG v9 (or Due).

There are two ways to program the SAM ARM processors that are on the TinyG v9 and Arduino Due: With a special hardware device called a "programmer," or just with the chips' built-in "bootloader."

If you are looking to program using a SAM-ICE from Atmel Studio 6, please see [here](https://github.com/synthetos/g2/wiki/Programming-v9-with-Studio6-and-the-SAM-ICE)

## Note about the SAM3X8x "bootloader"

The ROM of the SAM3X8x processors in use on the TinyG v9 and Arduino Due contains a program that will allow you to reprogram the flash in the processor.

Often microcontrollers have a program like this either built-in or installed and then left alone. During power-up or after a reset -- at "boot" -- the "bootloader" will run for a small amount of time (sometimes up to a few seconds). It that time, a specially-crafted program could talk to it and upload new firmware. If no such communication happens, then it will jump into the currently programmed firmware.

The "bootloader" on the SAM3Xxx processors, however, will NOT normally be run at boot time. Instead, the processor will start directly in the firmware. If, however, you have the ERASE pin held HIGH during boot (after power-on or a reset) then the firmware will be erased and a "Non-Volatile Memory" (NVM) bit will be set to boot into the ROM. Once this has been done, then you _must_ reprogram the flash.

An alternative method of getting into the "bootloader" -- by clearing the current firmware and setting the chip to boot from ROM -- is to connect to the board at 1200 baud and then immediately disconnect. (Note that the does not work with CoolTerm due to the way it sets the baud after connecting, and unsets it before disconnecting.)

From here on I will refer to is as a "loader" instead of a "bootloader."

The loader build in to the is the [SAM-BA® Boot Assistant](http://www.atmel.com/tools/atmelsam-bain-systemprogrammer.aspx).

## Programming with the ROM loader and BOSSAc

The TinyG v9 has a ERASE jumper, and the Due has an ERASE button. So, in order to put the board in the loader, you must either connect the ERASE jumper or hold the ERASE button _while_ hitting the Reset button. Then remove the ERASE jumper or release the ERASE button.
   
Please note: We do not yet have instructions Linux, but the steps are similar to that of OS X.

[Flashing TinyG2 in OSX](https://github.com/synthetos/g2/wiki/Flashing-TinyG2-with-Apple-OSX)<br>
[Flashing TinyG2 in Windows](https://github.com/synthetos/g2/wiki/Flashing-TinyG2-with-Windows)
