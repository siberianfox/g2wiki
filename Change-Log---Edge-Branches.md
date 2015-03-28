This page is a log of items updated in the edge branch and in the feature development branches cleaved off the main edge branch. Edge is moving pretty fast, and a number of people are working on projects, so we'll do our best to keep up with the changes here.

`Edge branch, build 079.60`
- Major changes to the way switches and other inputs are handled. See [Digital IO (GPIO)](Digital-IO-(GPIO)). The digital inputs are completed, the digital outputs have not been. In short, inputs are now just numbered inputs that are mapped to axes, functions, and hold behaviors. See settings/settings_shapeoko2.h for an example of setup and use. See also [Alarm Processing](Alarm-Processing).


Applies to `Edge branch, build 071.02`

* **No Persistence**. Most ARM chips (including the ATSAM3X8C on v9 and ATSAM3X8E on the Arduino Due) do not have persistence. This is the main reason the v9 has a microSD slot. But this has not been programmed yet. So your options are to either load the board each time you fire it up or reset it, or to build yourself a profile and compile your own settings as the defaults.

* **Still working on feedhold.** The serial communications runs a native USB on the ARM instead of through an FTDI USB-to-Serial adapter. We are still shing some bugs out of the single character commands such as feedhold (!), queue flush (%) and cycle start (~). 

* **Power Management needs work.** It doesn't always shut the motors off at the end of a cycle.

* **Different Behaviors**. There are some behaviors that are different.
  * Feedhold / queue flush on v8 works with !%~ in one line. In g2 it requires a newline. Use !\n%\n  This is due to using a USB stack that is partly on the chip and not being able to get at the individual characters that far upstream. This will probably not change in v9.
