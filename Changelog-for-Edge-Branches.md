This page is a log of items updated in the edge branch and in the feature development branches cleaved off the main edge branch. Edge is moving pretty fast, and a number of people are working on projects, so we'll do our best to keep up with the changes here.

https://github.com/synthetos/g2/README.md

###Edge branch, build 083.07
These changes are primarily fixes applied after testing
- Fixes to spindle speed settings (082.11)
- Fixes to build environments for Linux and other platforms
- Fixes to planner operation from edge-replan-replan
- Fixes for reporting error in inches mode

###Edge branch, build 082.10
These changes are still under test. If you find bugs or other issues please log to Issues.
- **[Digital IO (GPIO)](Digital-IO-(GPIO))** introduces major changes to the way switches and other inputs are handled. The digital inputs are completed, the digital outputs have not been. In short, inputs are now just numbered inputs that are mapped to axes, functions, and motion behaviors (feedholds). 
  - **Your configurations will need to change to accommodate these changes.** See settings/settings_shapeoko2.h for an example of setup and use - pay particular attention to `axis settings` and the new `inputs` section. 
  - Typing `$`, `$x`, `$di`, `$in` at the command line is also informative. Of course, all these commands are available as JSON, but in text mode you get the human readable annotations. 
  - These changes also rev the firmware version to 0.98 from 0.97, as a new configuration wiki page will need to be generated (not started yet).
  - {lim:0}, {lim:1} was added to allow a limit override to backing off switches when a limit is tripped
  - See also [Alarm Processing](Alarm-Processing), which is intimately related to these changes.

- **[Alarm processing](Alarm-Processing)** has been significantly updated. There are now 3 alarm states:
  - [ALARM](Alarm-Processing#alarm) - used to support soft and hard limits, safety interlock behaviors (door open), and other conditions.
  - [SHUTDOWN](Alarm-Processing#shutdown) - used to support external ESTOP functions (the controller doe NOT do ESTOP - read the SHUTDOWN section as to why.
  - [PANIC](Alarm-Processing#panic) - shuts down the machine immediately if there is an assertion failure or some other unrecoverable error
  - [CLEAR](Alarm-Processing#clear) describes how to clear alarm states.

- **[Job Exception Handling](Job-Exception-Handling)** has been refined. A new Job Kill has been introduced which is different than a queue flush (%), as these are actually 2 very different use cases.

- **Homing** changes. Homing input switches are now configured differently.
  - The switch configurations have been removed from the axes and moved to the digital IO inputs. 
  - Two new parameters have been added to the axis configs. All other parameters remain the same.
    - {xhd:1} - homing direction - 0=search-to-negative, 1=search-to-positive
    - {xhi:N} - homing input - 0=disable axis for homing, 1-N=enable homing for this input (switch) 
  - Note that setting the homing input to a non-zero value (1) enables homing for this axis, and (2) overrides whatever settings for that input for the duration of homing. So it's possible to set di1 (Xmin) as a limit switch and a homing switch. When not in homing it will be used as a limit switch.

- **Safety Interlock** added. 
  - An input configured for interlock will invoke a feedhold when the interlock becomes diseangaged and restart movement when re-engaged. 
  - {saf:0}, {saf:1} was added to enable or disable the interlock system.
  - There are optional settings for spindle and coolant actions on feedhold. See below

- **Spindle Changes** Expect updates to spindle behaviors in future branches. Here's where it is now:
  - The spindle can be paused on feedhold with the Spindle-pause-on-hold global setting {spph:1}. For now we recommend not using this {spph:0} as there is not yet a delay in spindle restart. 
  - Spindle enable and direction polarity can now be set using the {spep: } and {spdp: } commands.
  - Spindle enable and direction state can be returned using {spe:n} and {spd:n}, and these can be configured in status reports
  - Spindle speed can be returned using {sps:n} and can be configured in status reports

- **Coolant Changes** Expect coolant changes in future branches, in particular to accommodate changes in the digital outputs.
  - The coolant can be paused on feedhold with the Coolant-pause-on-hold global setting {coph:0}.
  - Flood and mist coolant polarity can now be set using the {cofp: } and {comp: } commands.
  - Flood and mist coolant state can be returned using {cof:n} and {com:n}, and these can be configured in status reports
  - In v9 the flood (M8) and mist (M7) commands are operative, but map the same pin. M9 clears them both, as expected. These should both be set to the same polarity for proper operation. On a Due or a platform with more output pins these can be separated - the code is written for this possibility. The changes should be limited to the pin mapping layers.

- **Power Management** is fully working, as far as we can tell. See $1pm for settings

- **Arc Changes** have been added. Please note any issues immediately. This is still under test.
 - Fixed bug on very large arcs 
 - Fixed bug on G18 rotation direction
 - Added P parameter to allow for arcs > 360 degree rotation

- **G10 L20** was added for easier offset setting

- **Bug Fixes**
  - Fixed some units mode display errors for G20 mode (inches)

- **Still To Go**
  - SD card persistence
  - Spindle restart dwell
  - Digital output generalization and changes
  - Still needs rigorous testing for very fast feedhold/resume and flush cycles

###Edge branch, build 071.02

* **No Persistence**. Most ARM chips (including the ATSAM3X8C on v9 and ATSAM3X8E on the Arduino Due) do not have persistence. This is the main reason the v9 has a microSD slot. But this has not been programmed yet. So your options are to either load the board each time you fire it up or reset it, or to build yourself a profile and compile your own settings as the defaults.

* **Still working on feedhold.** The serial communications runs a native USB on the ARM instead of through an FTDI USB-to-Serial adapter. We are still shing some bugs out of the single character commands such as feedhold (!), queue flush (%) and cycle start (~). 

* **Power Management needs work.** It doesn't always shut the motors off at the end of a cycle.

* **Different Behaviors**. There are some behaviors that are different.
  * Feedhold / queue flush on v8 works with !%~ in one line. In g2 it requires a newline. Use !\n%\n  This is due to using a USB stack that is partly on the chip and not being able to get at the individual characters that far upstream. This will probably not change in v9.