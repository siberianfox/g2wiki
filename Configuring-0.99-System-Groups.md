_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

# System Groups
The system groups contain the following global machine and communication settings. The system groups can be listed by requesting {sys:n} or $sys

- [Identification Parameters](#identification-parameters)
- [Global Machining Parameters](#global-machining-parameters)
- [Communications Parameters](#communications-parameters)
- [Gcode Initialization Defaults](#gcode-initialization-defaults)

##Identification Parameters
These are reported in the startup strings and should be included in any support discussions. All parameters are read-only unless otherwise noted.

	Setting | Description | Notes
	--------|-------------|-------
	[{fv:n}](#fvn-firmware-version) | Firmware Version | Version number, e.g. 0.99
	[{fb:n}](#fbn-firmware-build-number) | Firmware Build | Build number, e.g. 100.00
	[{fbs:n}](#fbsn-firmware-build-string) | Firmware Build String | Git string
	[{fbc:n}](#fbcn-firmware-build-config-file) | Firmware Build Config | Name of settings file 
	[{hp:n}](#hpn-hardware-platform) | Hardware_Platform | 1=Xmega, 2=Due, 3=v9(ARM)
	[{hv:n}](#hvn-hardware-version) | Hardware Version | Always set to 0 (used in v8 for HW configuration)
	[{id:n}](#idn-unique-board-identifier) | board ID | Each board has a read-only unique ID

### {fv:n} Firmware Version
Read-only value. Example `fv:0.99`. Indicates the major version of the firmware; changes infrequently. Generally all settings, behaviors and other system functions will remain the same within a version - that's why this page is useful for all 0.99 versions.

### {fb:n} Firmware Build number
Read-only value. Example `"fb":100.00`. 
Indicates the build of firmware and changes frequently. Please provide this number in any communication about an issue.

### {fbs:n} Firmware Build String
Git "describe" string / aka full version. Example `"fbs":"086.03-57-g843e-dirty"`. 
The format is "TAG-NN-sha1" with optional "-dirty", where NN is how many commits past the tag, and sha1 being the exact unique sha1 prefix. "-dirty" should not be seen unless you're doing development, and means there are uncommitted changes in the repo when it's built.

### {fbc:n} Firmware Build Config File
Filename of the settings file (config) complied for the build.<br>
Example `"fbc":"settings_makeblock.h"`

### {hp:n} Hardware Platform
Read-only value. Returns:
<pre>
{hv:1}  TinyG Xmega series
{hv:2}  Arduino Due G2 (ARM)
{hv:3}  TinyG v9 G2 (ARM)
</pre>

### {hv:n} Hardware Version
Unused in g2core. (This value is writable in TinyG and is used to set behaviors inside the firmware. Defaults to 8 for v8. If you have a TinyG v6 or earlier you must set this value to 6.)

### {id:n} Unique Board Identifier
Read-only value derived from factor configuration settings in the ARM chip


##Global Machining Parameters

	Setting | Description | Notes
	--------|-------------|-------
	[{jt:...}](#jt-junction-integration-time) | Junction Integration Time | Global cornering acceleration value
	[{ct:...}](#ct-chordal-tolerance) | Chordal Tolerance | Sets precision of arc drawing. Trades off precision for max arc draw rate 
	[{mt:...}](#mt-global-motor-power-timeout) | Motor_disable_Timeout | Number of seconds before power to motors is removed. Maximum value is 40 million seconds.

### {jt:...} Junction Integration Time 
The`{jt:...}` parameter is the way to set cornering velocity limits. JT is a normalized scaled factor that is nominally set to 1.000. Set to less than 1 for slower cornering (less aggressive), greater than 1 (but probably less than 2) for more aggressive cornering. This parameter replaces Junction Acceleration `{ja:...}` and the axis Junction Deviation commands - e.g. `{xjd:0.01}`.

<pre>
{jt:0.01}  Lower limit - probably will fail
{jt:0.75}  Good value for slower machines
{jt:1.00}  Nominal value
{jt:1.20}  Good value for fast machines
{jt:5.00}  Upper limit. Absolutely will fail
</pre>

If the JT parameter is set too low the machine will exhibit "stuttering" motion as mild corners will decelerate, possibly to zero. One way to tell if the setting it too low is to run a large arc, such as the following sequence: 

<pre>
G17 G21 G90 G91.1 G94
G10 L20 P2 X100 Y100 Z0
G55
G2 I50 F2000
</pre>

_Note: JT replaces the JA and xJD parameters in earlier builds. Cornering now obeys full jerk limitation instead of the centripetal acceleration heuristic, making it much more accurate and more true to the jerk limits set for the machine._

### {ct:...} Chordal Tolerance
Arcs are generated as sets of very short straight lines that approximate a curve. Each line is a "chord" that spans the endpoints of that segment of the arc. Chordal tolerance sets the maximum allowable deviation between the true arc and straight line that approximates it - which will be the value of the deviation in the middle of the line / arc.

Setting chordal tolerance high will make curves "rougher", but they can execute faster. Setting them smaller will make for smoother arcs that may take longer to execute. The lower-limit of $ct is set by the minimum arc segment length, which really should not be changed (See hidden parameters).
<pre>
{ct:0.01}  Normally a good value (in mm)
</pre>

### {mt:...} Global Motor Power Timeout
Sets the number of seconds motors will remain powered after the last 'event'. E.g. set to 60 to keep motors powered for 1 minute after a move completes. Only applies to motors with power management modes that actually time out the motors (modes 2 and 3). See also $ME and $MD commands, further down this page.
<pre>
{mt:5}        Keep motors energized for 5 seconds after last movement command
{mt:1000000}  Keep motors energized for 1 million seconds after last movement command (11.57 days)
</pre> 

See also [Power Management](Power-Management)

##Communications Parameters
Set communications speeds and modes. 

	Setting | Description | Notes
	--------|-------------|-------
	[{ej:...}](#ej-enable-json-mode) | Enable JSON mode | 0=text mode, 1=JSON mode, 2=auto mode
	[{jv:...}](#jv-json-verbosity) | JSON verbosity | 0=silent ... 5=verbose
	[{tv:...}](#tv-text-verbosity) | Text mode verbosity | 0=silent, 1=verbose
	[{qv:...}](#qv-queue-report-verbosity) | Queue report verbosity | 0=off, 1=filtered, 2=verbose
	[{sv:...}](#sv-status-report-verbosity) | Status_report_Verbosity | 0=off, 1=filtered, 2=verbose
	[{si:...}](#si-status-interval) | Status report interval | in milliseconds (100 ms minimum interval)
	[{ex:...}](#ex-flow-control) | Enable flow control | no function in 0.99 but remains for future use

_Note: As of 0.99 JSON syntax (`{js:n}`) has been removed. All responses are strict JSON format. G2core accepts commands in strict or relaxed JSON format_

### {ej:...} Enable JSON Mode
_Note: The behavior of EJ has changed in 0.99_<br><br>
Enable JSON mode sets the g2core response mode:
<pre>
{ej:0}  TEXT: Responses provided as Text (input commands are accepted in either format)
{ej:1}  JSON: Responses provided as JSON (input commands are accepted in either format)
{ej:2}  AUTO: Responses provided in the format of the request
</pre>

By selecting TEXT or JSON the setting is "sticky". Commands are accepted in either text or JSON mode, but responses are delivered in the selected mode. 

By selecting AUTO g2core will send responses in the same format as the command (it auto-configures according to the input). JSON mode is set by sending a line starting with an open curly '{'. TEXT mode is set by sending a line starting with '$', '?' or 'h'. Gcode commands do not change the auto mode. Responses to Gcode are returned in the current mode

_Note: The two startup lines on reset will always be in JSON format regardless of setting in order to allow UIs to sync with an unknown board._

### {jv:...} JSON verbosity
Sets how much information is returned in JSON mode. If you are using JSON mode with high-speed files (many short lines at high feed rates) you probably do not full verbose mode (5). 
<pre>
{jv:0}  Silent   - No response is provided for any command
{jv:1}  Footer   - Returns footer only - no command echo, gcode blocks or messages
{jv:2}  Messages - Returns footers, exception messages and gcode comment messages
{jv:3}  Configs  - Returns footer, messages, config command body
{jv:4}  Linenum  - Returns footer, messages, config command body, and gcode line numbers if present
{jv:5}  Verbose  - Returns footer, messages, config command body, and gcode blocks
</pre>

### {tv:...} Text Verbosity
Sets how much information is returned in text mode. We recommend using Verbose, except for very special cases.
<pre>
{tv:0}  Silent - no response is provided
{tv:1}  Verbose - returns OK and error responses
</pre>

### {qv:...} Queue Report Verbosity
Queue reports return the number of available buffers in the planner queue. The planner queue has 28 buffers and therefore can have as many as 28 Gcode blocks queued for execution. An empty queue will report 28 available buffers. A full one will report 0. 

Using the planner queue depth as a way to manage flow control when sending a Gcode file is actually a much better way than managing the serial input buffer. If you keep the planner full to about 2 blocks available it will run really smoothly. You also want to make sure the queue doesn't starve, say - more than 20 blocks available.

Verbosity settings are:
<pre>
{qv:0}  Silent - queue reports are off
{qv:1}  Single - returns reports when depth changes and is above hi water mark or below low water mark
{qv:2}  Triple - returns queue reports for every block queued to the planner buffer
</pre>

You can also get a manual queue report by sending [$qr](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration-for-Firmware-Version-0.97#qr---queue-report)

_Note: In general you don't want to fill up the buffer with more than 24 commands as some Gcode blocks can occupy multiple planner buffer slots (4 are allowed)._

### {sv:...} Status Report Verbosity
Please see [Status Reports](https://github.com/synthetos/TinyG/wiki/Status-Reports) for a discussion of $sv and $si status report settings.
<pre>
{sv:0}  Silent   - status reports are off
{sv:1}  Filtered - returns only changed values in status reports
{sv:2}  Verbose  - returns all values in status reports
</pre>

### {si:...} Status Interval 
The minimum is 100 ms. Trying to set a value below the minimum will set the minimum value. 
<pre>
{si:250}  Status interval in milliseconds
</pre>

### {ex:...} Enable Flow Control
This setting has no effect, but will not return an error if used.

Prior to 0.99 `ex` was used to select XON/XOFF, RTS/CTS or no flow control. In 0.99 there are two ways to communication o the board. (1) USB, which does its own flow control and therefore needs no configuration, and (2) UART serial which assumes (and currently requires) hardware RTS/CTS flow control. In the future we anticipate adding XON/XOFF software flow control back into the firmware, so this variable has not been removed. 


##Gcode Initialization Defaults
**NOTE: g2core currently does not support local persistence (i.e. on-board EEPROM or equivalent), so these Gcode initialization settings are effectively read-only. They can be set during compile time to change them, but a reset or power-on will always load the compiled values. Any changes that have been made to the values through the settings API are not applied.**  

Gcode settings loaded on power up and reset. Changing these does NOT change the current Gcode state, only the initialization settings. Being as how g2core does not have any native persistence these commands are only set at compile time and will not have an effect is written.

	Setting | Description | Notes
	--------|-------------|-------
	[{gpl:_}](#gpl-gcode-default-plane-selection) | PLane selection | 0=XY plane (G17), 1=XZ plane (G18), 2=YZ plane (G1)
	[{gun:_}](#gun-gcode-default-units) | UNits mode | 0=inches mode (G20), 1=mm mode (G21) 
	[{gco:_}](#gco-gcode-default-coordinate-system) | COordinate system | 1=G54, 2=G55, 3=G56, 4=G57, 5=G58, 6=G59
	[{gpa:_}](#gpa-gcode-default-path-control) | PAth_control_mode | 0=Exact path mode (G61), 1=Exact stop mode (G61.1), 2=Continuous mode (G64)
	[{gdi:_}](#gdi-gcode-distance-mode) | Distance mode | 0=Absolute mode (G90), 1=Incremental mode (G91)

These parameters set the values for the Gcode model on power-up or reset. They do not affect the current gcode dynamic model. 
For example, entering $gun=0 will cause the system to start up from reset or power up in inches mode, but will **not** change the system to inches mode when it is entered. A G20 or G21 received in the Gcode stream will change the units to inches or MM mode, respectively. On reset or restart they will change back to the $gun setting.

These parameters are reported as part of the "sys" group.

### {gpl:...} Gcode Default Plane Selection
<pre>
{gpl:0}  G17 (XY plane)
{gpl:1}  G18 (XZ plane)
{gpl:2}  G19 (YZ plane)
</pre> 

### {gun:...} Gcode Default Units
<pre>
{gun:0}  G20 (inches)
{gun:1}  G21 (millimeters)
</pre> 

### {gco:...} Gcode Default Coordinate System
<pre>
{gco:1}  G54 (coordinate system 1)
{gco:2}  G55 (coordinate system 2)
{gco:3}  G56 (coordinate system 3)
{gco:4}  G57 (coordinate system 4)
{gco:5}  G58 (coordinate system 5)
{gco:6}  G59 (coordinate system 6)
</pre> 

### {gpa:...} Gcode Default Path Control
<pre>
{gpa:0}  G61 (exact path mode)
{gpa:1}  G61.1 (exact stop mode)
{gpa:2}  G64 (continuous mode)
</pre> 

### {gdi:...} Gcode Distance Mode
<pre>
{gdi:0}  G90 (absolute mode)
{gdi:1}  G91 (incremental mode)
</pre> 
