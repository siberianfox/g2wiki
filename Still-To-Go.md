This is a list of things that currently are not finished or are known to not work quite right yet. Check back as this list will be updated.

Applies to `Edge branch, build 071.02`

* **No Persistence**. Most ARM chips (including the ATSAM3X8C on v9 and ATSAM3X8E on the Arduino Due) do not have persistence. This is the main reason the v9 has a microSD slot. But this has not been programmed yet. So your options are to either load the board each time you fire it up or reset it, or to build yourself a profile and compile your own settings as the defaults.

* **Still working on feedhold.** The serial communications runs a native USB on the ARM instead of through an FTDI USB-to-Serial adapter. We are still shing some bugs out of the single character commands such as feedhold (!), queue flush (%) and cycle start (~). 

* **Power Management needs work.** It doesn't always shut the motors off at the end of a cycle.