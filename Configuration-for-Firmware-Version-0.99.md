**The settings on this page are for firmware version 0.99.** 
The version number can be found as the fv variable in the startup JSON message, or by typing $fv. Version 0.99 encompasses builds 100.00 and later.

If you have an older firmware version:
* [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)

###Conventions Used for Config Documentation
- Examples show relaxed JSON input (e.g. `{fv:n}`). Strict JSON is also accepted in all cases (e.g. `{"fv":null}`)
- CMD means some command - aka the "name" of the name/value pair. CMDs are case insensitive.
- Underscore "_" means some numeric value
- "abcd" means some string value
- Motor examples use Motor 1, but any motor active for your platform is OK
- Axis examples use X axis, but any axis is OK unless otherwise noted
- All examples are in millimeter units

**A note about units**
- All data entry and display occurs in the prevailing units set in the Gcode mode. Set G20 for inches, G21 for mm. Query {unit:n} or $unit to find out where you are (G20=0, G21=1)
- Units entered in one system are correctly preserved and do not need to be re-entered or adjusted if the Gcode units mode changes.
- The Gcode default units setting `{gun:_}` sets the units the system "comes up in" during power-on or reset. So changing GUN will not change your units until you reset. Note: PROGRAM END (M2, M30) does not change the units. 

### JSON Cheat Sheet
All configuration commands are available using [JSON mode](JSON-Operation), which is the preferred access method if you are writing a UI or controller. Most commands are also available in [Text Mode](Text-Mode). The few commands that are available from only one or the other are noted, as are any commands the behave differently depending on the mode. The rough equivalence is:
<pre>
{"CMD":n} == $CMD            Read value for command "CMD" in strict JSON mode
{CMD} == $CMD                Read value in relaxed JSON mode
{"CMD":123.4} == $CMD=123.4  Set value to 123.4 in strict JSON mode
{CMD:123.4} == $CMD=123.4    Set value in relaxed JSON mode
{xvm:50000}                  Set X max velocity to 50000 as an example
</pre>
You can also use JSON to read an entire object - examples for motor and axis:
<pre>
{1:n}
{"r":{"1":{"ma":0,"sa":1.800,"tr":40.0000,"mi":8,"po":0,"pm":2,"pl":0.375}},"f":[1,0,5]}

{x:n}
{"r":{"x":{"am":1,"vm":50000,"fr":50000,"tn":0.000,"tm":420.000,"jm":10000,"jh":20000,"jd":0.1000,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":20.000,"zb":3.000}},"f":[1,0,5]}
</pre>
The footer is an array of 3 elements:
- (1) Footer revision
- (2) [Status code](Status-Codes) - Short version: status=0 is OK, everything else is an exception.
- (3) RX buffer info (explained later)
- Note: The checksum that used to be in the footer has been removed as no-one was using it.

# Configuration Pages

- [System Configuration Group](Configuration 0.99 System Group)
- [Motor Configuration](Configuration 0.99 Motor Group)
- [Axis Configuration](Configuration 0.99 Axis Group)
- [Other Groups](Configuration 0.99 Axis Groups)
- [Actions and Reports](Configuration 0.99 Actions and Reports)
- [3DP Extensions Configuration](Configuration 0.99 3DP Extensions)



## System Group
The system group contains the following global machine and communication settings. The system group can be listed by requesting {sys:n} or $sys

**Identification Settings**
These are reported on the startup strings and should be included in any support discussions.

	Setting | Description | Notes
	--------|-------------|-------
	[{fv:n}](#fv---firmware-version) | Firmware Version | Read-only value, e.g. 0.99
	[{fb:n}](#fb---firmware-build-number) | Firmware Build | Read-only value, e.g. 100.00
	[{fbs:n}](#fb---firmware-build-string) | Firmware Build String | Read-only string
	[{fbc:n}](#fb---firmware-build-config) | Firmware Build Config | Read-only filename
	[{hp:n}](#hp---hardware-platform) | Hardware_Platform | Read-only value, 1=Xmega, 2=Due, 3=v9(ARM)
	[{id:n}](#id---unique-board-identifier) | board ID | Each board has a read-only unique ID
	[{hv:_}](#hv---hardware-version) | Hardware Version | **V8 only** Set to 6 for v6 and earlier boards, 7 or 8 for v7 and v8 boards. Defaults to 8. 

**Global System Settings**

	Setting | Description | Notes
	--------|-------------|-------
	[{jt:_}](#ja---junction-integration-time) | Junction Integration Time | Global cornering acceleration value
	[{ct:_}](#ct---chordal-tolerance) | Chordal Tolerance | Sets precision of arc drawing. Trades off precision for max arc draw rate 
	[{mt:_}](#mt---motor-power-timeout) | Motor_disable_Timeout | Number of seconds before motor power is automatically released. Maximum value is 40 million.

**Communications Settings**
Set communications speeds and modes. 

	Setting | Description | Notes
	--------|-------------|-------
	[{ej:_}](#ej---enable-json-mode-on-power-up) | Enable JSON mode | 0=text mode, 1=JSON mode
	[{jv:_}](#jv---set-json-verbosity) | JSON verbosity | 0=silent ... 5=verbose
	[{js:_}](#js---set-json-syntax) | JSON syntax | 0=relaxed, 1=strict
	[{tv:_}](#tv---set-text-mode-verbosity) | Text mode verbosity | 0=silent, 1=verbose
	[{qv:_}](#qv---queue-report-verbosity) | Queue report verbosity | 0=off, 1=filtered, 2=verbose
	[{sv:_}](#sv---status-report-verbosity) | Status_report_Verbosity | 0=off, 1=filtered, 2=verbose
	[{si:_}](#si---status-interval) | Status report interval | in milliseconds (100 ms minimum interval)

V8-Only Communications Settings (not available on g2)

	Setting | Description | Notes
	--------|-------------|-------
	[{ec:_}](#ec---expand-lf-to-crlf-on-tx-data) | Enable CR on TX | 0=send LF line termination on TX, 1= send both LF and CR termination
	[{ee:_}](#ee---enable-character-echo) | Enable_character_Echo | 0=off, 1=enabled
	[{ex:_}](#ex---enable-flow-control) | Enable flow control | 0=off, 1=XON/XOFF enabled, 2=RTS/CTS enabled
	[{baud:_}](#baud---set-usb-baud-rate) | Baud rate | 1=9600, 2=19200, 3=38400, 4=57600, 5=115200, 6=230400 -- 115200 is default


**Gcode Initialization Defaults**
Gcode settings loaded on power up and reset. Changing these does NOT change the current Gcode state, only the initialization settings. 

	Setting | Description | Notes
	--------|-------------|-------
	[{gpl:_}](#gpl---gcode-default-plane-selection) | PLane selection | 0=XY plane (G17), 1=XZ plane (G18), 2=YZ plane (G1)
	[{gun:_}](#gun---gcode-default-units) | UNits mode | 0=inches mode (G20), 1=mm mode (G21) 
	[{gco:_}](#gco---gcode-default-coordinate-system) | COordinate system | 1=G54, 2=G55, 3=G56, 4=G57, 5=G58, 6=G59
	[{gpa:_}](#gpa---gcode-default-path-control) | PAth_control_mode | 0=Exact path mode (G61), 1=Exact stop mode (G61.1), 2=Continuous mode (G64)
	[{gdi:_}](#gdi---gcode-distance-mode) | Distance mode | 0=Absolute mode (G90), 1=Incremental mode (G91)

##Commands and Reports
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
# Settings Details

## System Group Settings
These are general system-wide parameters and are part of the "sys" group.
<br>
###Identification Settings

### $FV - Firmware Version
Read-only value. Example `$fv=0.97`<br>
Indicates the major version of the firmware; changes infrequently. Generally all settings, behaviors and other system functions will remain the same within a version - that's why this page is useful for all 0.97 versions.

### $FB - Firmware Build number
Read-only value. Example `$fb=345.06`<br>
Indicates the build of firmware and changes frequently. Please provide this number in any communication about an issue.

### $FBS - Firmware Build String
Read-only value. Example `$fb=345.06`<br>
Provides the git string for the build

### $FBC - Firmware Build Config
Read-only value. Example `$fb=345.06`<br>
Provides the filename of the settings file (config) complied for the build

### $HP - Hardware Platform
Read-only value. Returns:
* 1 = TinyG Xmega series
* 2 for Arduino Due G2 (ARM)
* 3 for TinyG v9 G2 (ARM)

### $HV - Hardware Version
Read-write value. Used to set behaviors inside the firmware. Defaults to 8 for v8. If you have a TinyG v6 or earlier you must set this value to 6.

### $ID - Unique Board Identifier
Read-only value.

###Global System Settings

### $JA - Junction Acceleration 
In conjunction with the global $jd setting sets the cornering speed. See $jd for explanation

<pre>
$ja=50000   - 50,000 mm/min^2 - a reasonable value for a modest performance machine
$ja=200000  - 200,000 mm/min^2 - a reasonable value for a higher performance machine
</pre> 

### $CT - Chordal Tolerance
Arcs are generated as sets of very short straight lines that approximate a curve. Each line is a "chord" that spans the endpoints of that segment of the arc. Chordal tolerance sets the maximum allowable deviation between the true arc and straight line that approximates it - which will be the value of the deviation in the middle of the line / arc.

Setting chordal tolerance high will make curves "rougher", but they can execute faster. Setting them smaller will make for smoother arcs that may take longer to execute. The lower-limit of $ct is set by the minimum arc segment length, which really should not be changed (See hidden parameters).
<pre>
$ct=0.01   - Normally a good value (in mm)
</pre> 

### $ST - Switch Type
Sets the type of switch used for homing and/or limits. All switches must be of the same type (mixes are not supported).
<pre>
$st=0   - Normally Open switches (NO)
$st=1   - Normally Closed switches (NC)
</pre> 

Note that probing cycles (G38.2) will work regardless of the switch setting. Probing (currently) assumes a normally open switch in the Z minimum position. During the probe cycle switches are set to NO (and ignored). They are restored to their actual $st setting when probing is complete.

### $MT - Motor Power Timeout
Sets the number of seconds motors will remain powered after the last 'event'. E.g. set to 60 to keep motors powered for 1 minute after a move completes. Only applies to motors with power management modes that actually time out the motors (modes 2 and 3). See also $ME and $MD commands, further down this page.
<pre>
$mt=5        - Keep motors energized for 5 seconds after last movement command
$mt=1000000  - Keep motors energized for 1 million seconds after last movement command (11.57 days)
</pre> 

See also [Power Management](Power-Management)
###Communications Settings

### $EJ - Enable JSON Mode on Power Up
This sets the startup mode. JSON mode can be invoked at any time by sending a line starting with an open curly '{'. JSON mode is exited any time by sending a line starting with '$', '?' or 'h'

_Note: The two startup lines on reset will always be in JSON format regardless of setting in order to allow UIs to sync with an unknown board._

<pre>
$ej=0      - Disable JSON mode on power-up and reset
$ej=1      - Enable JSON mode on power-up and reset
</pre>

### $JV - Set JSON verbosity
Sets how much information is returned in JSON mode. If you are using JSON mode with high-speed files (many short lines at high feed rates) you probably do not full verbose mode (5). 
<pre>
$jv=0      - Silent   - No response is provided for any command
$jv=1      - Footer   - Returns footer only - no command echo, gcode blocks or messages
$jv=2      - Messages - Returns footers, exception messages and gcode comment messages
$jv=3      - Configs  - Returns footer, messages, config command body
$jv=4      - Linenum  - Returns footer, messages, config command body, and gcode line numbers if present
$jv=5      - Verbose  - Returns footer, messages, config command body, and gcode blocks
</pre>

### $JS - Set JSON syntax
Sets relaxed or strict syntax for JSON messages. See [JSON Syntax](JSON-Operation#json-syntax-option---relaxed-or-strict) for details.

### $TV - Set Text mode verbosity
Sets how much information is returned in text mode. We recommend using Verbose, except for very special cases.
<pre>
$tv=0      - Silent - no response is provided
$tv=1      - Verbose - returns OK and error responses
</pre>

### $QV - Queue Report Verbosity
Queue reports return the number of available buffers in the planner queue. The planner queue has 28 buffers and therefore can have as many as 28 Gcode blocks queued for execution. An empty queue will report 28 available buffers. A full one will report 0. 

Using the planner queue depth as a way to manage flow control when sending a Gcode file is actually a much better way than managing the serial input buffer. If you keep the planner full to about 2 blocks available it will run really smoothly. You also want to make sure the queue doesn't starve, say - more than 20 blocks available.

Verbosity settings are:
<pre>
$qv=0      - Silent - queue reports are off
$qv=1      - Single - returns reports when depth changes and is above hi water mark or below low water mark
$qv=2      - Triple - returns queue reports for every block queued to the planner buffer
</pre>

You can also get a manual queue report by sending [$qr](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration-for-Firmware-Version-0.97#qr---queue-report)

_Note: In general you don't want to fill up the buffer with more than 24 commands as some Gcode blocks can occupy multiple planner buffer slots (4 are allowed)._

### $SV - Status Report Verbosity
Please see [Status Reports](https://github.com/synthetos/TinyG/wiki/Status-Reports) for a discussion of $sv and $si status report settings.
<pre>
$sv=0      - Silent   - status reports are off
$sv=1      - Filtered - returns only changed values in status reports
$sv=2      - Verbose  - returns all values in status reports
</pre>

### $SI - Status Interval 
The minimum is 100 ms. Trying to set a value below the minimum will set the minimum value. 
<pre>
$si=250    - Status interval in milliseconds
</pre>

### $IC - Ignore CR or LF on RX 
_Note: IC was removed in 0.97. Lines may be terminated with any combination of <CR>, <LF>, <CR<LF>, <LF<CR>. It doesn't matter anymore._

### $EC - Expand LF to CRLF on TX data
If this setting is OFF returned lines have a single <LF>. If on they are terminated with <CR><LF> (2 characters).
<pre>
$ec=0      - off
$ec=1      - on
</pre> 

### $EE - Enable Character Echo 
This should be disabled for JSON mode. In text mode it's optional either way.
<pre>
$ee=0      - Disable character echo
$ee=1      - Enable character echo
</pre> 

### $EX - Enable Flow Control 
<pre>
$ex=0      - Disable flow control 
$ex=1      - Enable XON/XOFF flow control protocol 
$ex=2      - Enable RTS/CTS flow control protocol 
</pre>

### $BAUD - Set USB Baud Rate
The default baud rate for the USB port is 115,200 baud. The following additional baud rates may be set. The sequence for changing the baud rate is: (1) Issue the $baud command, (2) wait for a response verifying the command, (3) change to the new baud rate.
<pre>
$baud=0     - Illegal baud rate setting. Returns an error
$baud=1     - 9600
$baud=2     - 19200
$baud=3     - 38400
$baud=4     - 57600
$baud=5     - 115200
$baud=6     - 230400
</pre>

####Gcode Default Parameters
These parameters set the values for the Gcode model on power-up or reset. They do not affect the current gcode dynamic model. <br><br>
For example, entering $gun=0 will cause the system to start up from reset or power up in inches mode, but will **not** change the system to inches mode when it is entered. A G20 or G21 received in the Gcode stream will change the units to inches or MM mode, respectively. On reset or restart they will change back to the $gun setting.

These parameters are reported as part of the "sys" group.

### $GPL - Gcode Default Plane Selection
<pre>
$gpl=0      - G17 (XY plane)
$gpl=1      - G18 (XZ plane)
$gpl=2      - G19 (YZ plane)
</pre> 

###$GUN - Gcode Default Units
<pre>
$gun=0      - G20 (inches)
$gun=1      - G21 (millimeters)
</pre> 

###$GCO - Gcode Default Coordinate System
<pre>
$gco=1      - G54 (coordinate system 1)
$gco=2      - G55 (coordinate system 2)
$gco=3      - G56 (coordinate system 3)
$gco=4      - G57 (coordinate system 4)
$gco=5      - G58 (coordinate system 5)
$gco=6      - G59 (coordinate system 6)
</pre> 

###$GPA - Gcode Default Path Control
<pre>
$gpa=0      - G61 (exact path mode)
$gpa=1      - G61.1 (exact stop mode)
$gpa=2      - G64 (continuous mode)
</pre> 

### $GDI - Gcode Distance Mode
<pre>
$gdi=0      - G90 (absolute mode)
$gdi=1      - G91 (incremental mode)
</pre> 

## Coordinate System and Origin Offsets 
### $g54x - $g59c
Coordinate system offsets are the values used by G54, G55, G56, G57, G58 and G59 commands to define the offsets from the machine (absolute) coordinate system for X,Y,Z,A,B and C. G54-G59 correspond to the Gcode coordinate systems 1-6, respectively. 

By convention G54 is often set to no offsets (all zeroes) so it is the same as the machine's absolute coordinate system. This is true because the G53 command "move in absolute coordinates" is only in effect for the current Gcode block. After that the dynamic model reverts to the coordinate system previously in effect. So if you want to say in absolute coordinates you need a persistent machine coordinate system, by convention G54.

Another convention is to set G55 to your common coordinate system, we set this to be 0,0 in the middle of the table. So once you have zeroed issuing g55 g28 will set to this system and position the head in the middle of the table. (Note: this can be done on one line of gcode - it does not need to be 2 separate commands).

G54-G59 offsets can be set per the following example:
<pre>
$g54x=0         Set G54 to be the same as the machine coordinate system
$g54y=0
$g54z=0
$g54a=0
$g54b=0
$g54c=0

$g55x=90.0      Set G55 to be in the middle of the table
$g55y=90.0
$g55z=0
$g55a=0
$g55b=0
$g55c=0
</pre> 

In JSON mode you can set a coordinate system in a single command. Only those axes specified are changed. 
<pre>
{"g55"":{"x":90,"y":90,"z":"0"}}
</pre> 

#### Displaying Offsets
Offsets can be displayed individually

`$g54x` - returns a single value 

...or as a group: 
`$g54` - returns all 6 values in the G54 group 
`$g92` - returns all 6 values of the origin offset group 

...or all together: 
`$o` - returns all offsets in the system (not available in JSON)

Note: the G54-G59 settings are persistent settings that are preserved between resets (i.e. in EEPROM), unlike the G92 origin offset settings and the G28 and G30 "go home" settings which are just in the volatile Gcode model and are thus not preserved. 

#### G10 Operation
Gcode provides the G10 L2 command to perform this same coordinate system offset setting function. Coordinate offsets can be set from Gcode using the G10 command, e.g. G10 P2 L2 X20.000 - the P word is the coordinate system numbered 1-6, the L word =2 is according to standard, but is ignored by TinyG (for now)

TinyG does not persist G10 settings, however. This is not in accordance with the [Gcode spec](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CEkQFjAA&url=http%3A%2F%2Fciteseerx.ist.psu.edu%2Fviewdoc%2Fdownload%3Fdoi%3D10.1.1.141.2441%26rep%3Drep1%26type%3Dpdf&ei=rm7UULm2F6ex0AH1sICQDA&usg=AFQjCNH72m_sRg2TycD-8cf8d40FZ6Co_g&sig2=MrjWabHA5YwPsWtrj2TtOA&bvm=bv.1355534169,d.dmQ). Any G10 settings that are provided will be used until reset, power cycle, or they are overwritten by a $g5xx command or another G10 command. 

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

## Hidden Parameters
These parameters are not part of any group and generally should not be changed. Serious malfunction can occur if these are not set correctly

### $ML- Minimum Line Segment 
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>
$ml=0.08    - Do not change this value
</pre> 

### $MA - Minimum Arc Segment 
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>$ma=0.10    - Do not change this value
</pre> 

### $MS - Minimum Segment time in microseconds - Refers to S-curve interpolation segments
Don't change this unless you are seriously tweaking TinyG for your application. It can cause many things to break. This value does not appear in system group listings ($sys)
<pre>
$ms=5000  - Do not change this value
</pre> 