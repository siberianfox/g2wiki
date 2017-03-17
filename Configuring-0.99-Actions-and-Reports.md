_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

## Commands and Reports
These $configs invoke reports and functions

Command | Description | Notes
--------|-------------|-------
[{sr:n}](#sr---status-report) | get Status Report | SR also sets status report format in JSON mode
[{qr:n}](#qr---queue-report) | get Queue Report | 
[{qf:_}](#qf---queue-flush) | flush_planner_queue | Used with '!' feedhold for jogging, probes and other sequences. Usage: {qf:1}
[{md:n}](Power-Management) | Disable motors | Unpower all motors
[{me:_}](Power-Management) | Energize motors | Energize all motors with timeout in seconds 
[{test:_}](#test---run-self-test) | Invoke self tests | $test=n for test number; $test returns help screen in text mode
[{defa:1}](#defa---reset-default-profile-settings) | reset to DEFAults | Reset to compiled default settings
[{flash:1}] | Enter_FLASH_loader | WILL ERASE AND REFLASH BOARD. BE CAREFUL WITH THIS
^x | Reset tinyG | cntl+x restarts tinyG same as hardware rest button
[{help:_}] | Show help screen | Show system help screen; $h also works

Note: Status report parameters is settable in JSON only - see JSON mode for details
<br>

## Commands
These commands cause various actions, and are not technically "settings".

### $SR - Status Report
Returns a status report or set the contents of a status report (JSON only). Identical to ? command. See [Status Reports]() for details.

### $QR - Queue Report
Manually request a queue report. See [$QV](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration#qv---queue-report-verbosity) for details.

### $QF - Queue Flush
Removes all Gcode blocks remaining in the planner queue. This is useful to clear the buffer after a feedhold to create homing, jogging, probes and other cycles.

### $MD - Motor De-energize
Unpower all motors. See [Power Management](Power-Management)

### $ME - Motor Energize
Energizes all non-disabled motors. Motors will time out according to their power management a mode and timeout value. See [Power Management](Power-Management)

### $TEST - Run Self Test
Execute `$test` to get a listing of available tests. Run `$test=N`, where N is the test number.

### $DEFA - Reset default profile settings
TinyG comes with a set of defaults pre-programmed to a specific machine profile. The default profile is set for a relatively slow screw machine such as the Zen Toolworks 7x12. Other default profiles are settable at compile time by including the right settings_XXXXXX.h file. If you are having trouble with your settings and want to revert to the default settings enter: `$defa=1`  

***WARNING*** This will revert all settings to defaults. Do a screencap of the $$ dump if you want to refer back to the current settings
