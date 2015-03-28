This page is a log of items updated in the edge branch and in the feature development branches cleaved off the main edge branch. Edge is moving pretty fast, and a number of people are working on projects, so we'll do our best to keep up with the changes here.

###Edge branch, build 079.60
These changes are still under test. If you find bugs or other issues please log to Issues.
- [Digital IO (GPIO)](Digital-IO-(GPIO)) introduces major changes to the way switches and other inputs are handled. The digital inputs are completed, the digital outputs have not been. In short, inputs are now just numbered inputs that are mapped to axes, functions, and motion behaviors (feedholds). 
  - **Your configurations will need to change to accommodate these changes.** See settings/settings_shapeoko2.h for an example of setup and use - pay particular attention to `axis settings` and the new `inputs` section. 
  - Typing `$`, `$x`, `$di`, `$in` at the command line is also informative. Of course, all these commands are available as JSON, but in text mode you get the human readable annotations. 
  - These changes also rev the firmware version to 0.98 from 0.97, as a new configuration wiki page will need to be generated (not started yet). 
  - See also [Alarm Processing](Alarm-Processing), which is intimately related to these changes.

- [Alarm processing](Alarm-Processing) has been significantly updated. There are now 3 alarm states:
  - [ALARM](Alarm-Processing#alarm) - used to support soft and hard limits, safety interlock behaviors (door open), and other conditions.
  - [SHUTDOWN](Alarm-Processing#shutdown) - used to support external ESTOP functions (the controller doe NOT do ESTOP - read the SHUTDOWN section as to why.
  - [PANIC](Alarm-Processing#panic) - shuts down the machine immediately if there is an assertion failure or some other unrecoverable error
  - [CLEAR](Alarm-Processing#clear) describes how to clear alarm states.

- [Job Exception Handling](Job-Exception-Handling) has been refined. A new Job Kill has been introduced which is different than a queue flush (%), as these are actually 2 very different use cases.

- **Homing** changes. Homing input switches are now configured differently.
  - The switch configurations have been removed from the axes and moved to the digital IO inputs. 
  - Two new parameters have been added to the axis configs (see below). All other parameters remain the same.
    - {xhd:1} - homing direction - 0=search-to-negative, 1=search-to-positive
    - {xhi:N} - homing input - input 1-N or 0 to disable homing this axis for homing. Note that setting this to a non-zero valaue (1) enables homing for this axis, and (2) overrides whatever settings for that input for the duration of homing. So it's possible to set di1 (Xmin) as a limit switch and a homing switch. Whenn not in homing it will be used as a limit switch.

###Edge branch, build 071.02

* **No Persistence**. Most ARM chips (including the ATSAM3X8C on v9 and ATSAM3X8E on the Arduino Due) do not have persistence. This is the main reason the v9 has a microSD slot. But this has not been programmed yet. So your options are to either load the board each time you fire it up or reset it, or to build yourself a profile and compile your own settings as the defaults.

* **Still working on feedhold.** The serial communications runs a native USB on the ARM instead of through an FTDI USB-to-Serial adapter. We are still shing some bugs out of the single character commands such as feedhold (!), queue flush (%) and cycle start (~). 

* **Power Management needs work.** It doesn't always shut the motors off at the end of a cycle.

* **Different Behaviors**. There are some behaviors that are different.
  * Feedhold / queue flush on v8 works with !%~ in one line. In g2 it requires a newline. Use !\n%\n  This is due to using a USB stack that is partly on the chip and not being able to get at the individual characters that far upstream. This will probably not change in v9.
