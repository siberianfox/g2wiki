This page describes the text mode command line in G2. Text mode is intended as a human readable way to access the system and perform simple tasks. For UI and machine-to-machine connections we recommend using JSON mode.

## Text Mode and JSON Mode
G2 can operate in either text mode (command line mode) or JSON mode. In text mode G2 accepts $ config lines and normal Gcode blocks, and returns responses as human-friendly ASCII text. Example:
 
	Command | Description
	--------|--------------
	$xvm | View X axis maximum velocity
	$xvm=16000 | Set X axis maximum velocity


G2 can also operate using JSON commands. This is the preferred way to drive G2 from a UI or controller. Using JSON dramatically simplifies the UI as the board can be treated as a RESTful resource, and no parsers or special handlers need to be written. Example:

	Command | Description
	--------|--------------
	{"xvm":n} | View X axis maximum velocity
	{"xvm":16000} | Set X axis maximum velocity
	{xvm:16000} | Relaxed JSON mode equivalent to above

See [JSON Operation](JSON-Operation) and [JSON Details](JSON-Details).

_Note: JSON examples in this page are in relaxed JSON mode to void typing all those quotes_

### Startup Modes
G2 starts up in text mode if the $ej setting is set to text mode ($ej=0). G2 will also enter text mode automatically if it receives a line with a leading $, ? or 'h'. 

G2 starts up in JSON mode if the $ej setting is set to JSON mode ($ej=1). G2 will also enter JSON mode automatically if it receives a line starting with an open curly '{'. While in JSON mode all commands are expected in JSON format and responses are returned in JSON format (except for special handling of [Gcode in JSON]()).

**Note: The initial status messages returned on bootup will be in JSON format, regardless of the mode set in order to not break JSON parsers during the startup sequence.**


***
**The rest of this page covers things that are common to both text and JSON operation.**

## Names and Tokens 
Names are short mnemonic tokens that can be 1 to 5 characters in length. Axis and motor tokens are typically 3 characters including their axis or motor prefix; Non-axis and non-motor (general) tokens are 2 to 5 characters. 

Tokens are case insensitive and can only contain alphanumeric characters. See [G2 Configuration](Configuration-for-Firmware-Version-0.98) for a complete list of the tokens used for settings. Some examples are provided below: 

	Token | as Text | as JSON | Description
	------|---------|---------|--------------
	xfr | $xfr | {"xfr":""} | X axis maximum feed rate
	cjm | $cjm | {"cjm":""} | C axis maximum jerk 
	1mi | $1mi | {"1mi":""} | Motor 1 microstep setting 
	home | $home | {"home":""} | Homing state
	x | $x | {"x":""} | X axis group. In text mode a group can only be queried (get). In JSON mode a group can be queried and can also be used to set any or all values in the group

## Groups
A group is a collection of related tokens. Groups are used to specify all parameters for a motor, an axis, a heater, a PWM channel, or some other logical grouping. A group is similar in concept to a RESTful resource or composite. Groups simplify configuration management and reporting by collecting related values together. The following groups are defined.

	Group | Tokens | Notes
	--------|----------|-------
	system | sys | system global parameters
	axis | x y x a b c | all settings for that axis
	motor | 1 2 3 4 5 6 | all settings for that motor
	pwm | p1...pN | all settings for pulse width modulation channel
	offsets | g54 g55 g56 g57 g58 g59 | offset settings xyzabc in named coordinate system. 
	temporary offset | g92 | g92 offset settings. These are not persistent
	return to home values | g28, g30 | return values in machine coordinates
	pos | x y x a b c | current work position; with offsets in current units
	mpo | x y x a b c | current machine position; no offsets, always in millimeters
	ofs | x y x a b c | current offsets,  always in millimeters
	hom | x y x a b c e | homing status by axis. 'e' reports homing status for Entire machine
	prb | x y x a b c e | probe state by axis. 'e' reports success or failure
	di | 1...N | digital input configuration
	in | 1...N | digital input readouts (switch readers)
	do | 1...N | digital output configuration
	out | 1...N | digital output readouts
	pid | 1 2 3 | PID configuration
	he | 1 2 3 | heater configuration and readouts

To list a group type in the group prefix; for example:
* type `$x` to list the X group (or `{x:n}` in JSON)
* type `$sys` to list the system group (or simply `$`, which is text shorthand, or `{sys:n}` )
* type `$g55` to list the xyzabc offsets in the G55 coordinate system (or `{g55:n}` )

**Diagnostic Groups**<br>
In addition the the operating groups above a series of diagnostic groups can be enabled at compile time See [Diagnostics](Diagnostics)

### Uber-Groups
In text mode the following groups of groups are also available for display:

	Uber-Group | Token | Notes
	--------|----------|-------
	axis | q | All axis settings for all axes
	motor | m | All settings for all motors
	offsets | o | All offset settings including g92's
	all | $ | All settings. Invoked as $$

For example type `$q` to list all axis groups. 
The list of uber-groups can be seen by asking for the system help screen using `$h`.

In JSON mode these groups will return 1 or more `{r:}` responses with one group in each response. This is not the correct way for G2 to handle this command. For example, `{q:n}` should return a response with a parent q like so: `{r:{q:{1:{...}}}}` with the groups as children. We plan to correct this in a future release.

## Displaying Settings and Groups
When displaying or setting configs a `$` must be the first character of the line. Input is case insensitive.

_**A note about units**_
_Settings can be displayed and entered in either inches or millimeters. All values entered and responses provided will be in the current Gcode UNITS setting: `G20` for inches or `G21` for mm. Most of the examples below are in mm, but could just as easily be input in inches._

To display a setting type $<the-mnemonic-for-the=setting-you-want-to-display>. It will respond with the value taken. For example:

	Request | Response | Notes
	--------|----------|-------
	$xvm | [xvm] x_velocity_maximum 16000.000 mm/min | Show X axis maximum velocity
	$3po | [3po] m3_polarity 0 [0,1] | Show motor 3 polarity
	$ex | [ex] enable_xon_xoff 1 [0,1] | Show XON/XOFF setting

The following commands will display groups.
<pre>
$x    --- Show all X axis settings ---
[xam] x axis mode                 1 [standard]
[xvm] x velocity maximum      16000.000 mm/min
[xfr] x feedrate maximum      16000.000 mm/min
[xtm] x travel maximum          300.000 mm
[xjm] x jerk maximum     5000000000 mm/min^3
[xjh] x jerk homing     10000000000 mm/min^3
[xjd] x junction deviation        0.0100 mm (larger is faster)
[xsn] x switch min                1 [0=off,1=homing,2=limit,3=limit+homing]
[xsx] x switch max                0 [0=off,1=homing,2=limit,3=limit+homing]
[xsv] x search velocity        3000.000 mm/min
[xlv] x latch velocity          100.000 mm/min
[xlb] x latch backoff            20.000 mm
[xzb] x zero backoff              3.000 mm
tinyg [mm] ok> 
</pre>

<pre>
$x    --- Show all X axis settings in inches mode (issue g20 beforehand) ---
[xam] x axis mode                 1 [standard]
[xvm] x velocity maximum        629.921 in/min
[xfr] x feedrate maximum        629.921 in/min
[xtm] x travel maximum           11.811 in
[xjm] x jerk maximum      196850400 in/min^3
[xjh] x jerk homing       393700800 in/min^3
[xjd] x junction deviation        0.0004 in (larger is faster)
[xsn] x switch min                1 [0=off,1=homing,2=limit,3=limit+homing]
[xsx] x switch max                0 [0=off,1=homing,2=limit,3=limit+homing]
[xsv] x search velocity         118.110 in/min
[xlv] x latch velocity            3.937 in/min
[xlb] x latch backoff             0.787 in
[xzb] x zero backoff              0.118 in
</pre>

<pre>
$3    --- Show all motor 3 settings ---
[3ma] m3 map to axis              2 [0=X,1=Y,2=Z...]
[3sa] m3 step angle               1.800 deg
[3tr] m3 travel per revolution    1.250 mm
[3mi] m3 microsteps               4 [1,2,4,8]
[3po] m3 polarity                 1 [0=normal,1=reverse]
[3pm] m3 power management         1 [0=off,1=on]
</pre>

<pre>
$sys  --- Show all system settings ---
[fb]  firmware build            377.08
[fv]  firmware version            0.95
[hv]  hardware version            7.00
[id]  TinyG ID                    2X2660-FHZ
[ja]  junction acceleration 2000000 mm
[ct]  chordal tolerance           0.001 mm
[st]  switch type                 1 [0=NO,1=NC]
[ej]  enable json mode            0 [0=text,1=JSON]
[jv]  json verbosity              2 [0=silent,1=footer,2=messages,3=configs,4=linenum,5=verbose]
[tv]  text verbosity              1 [0=silent,1=verbose]
[qv]  queue report verbosity      0 [0=off,1=filtered,2=verbose]
[sv]  status report verbosity     1 [0=off,1=filtered,2=verbose]
[si]  status interval           250 ms
[ic]  ignore CR or LF on RX       0 [0=off,1=CR,2=LF]
[ec]  expand LF to CRLF on TX     0 [0=off,1=on]
[ee]  enable echo                 0 [0=off,1=on]
[ex]  enable xon xoff             1 [0=off,1=on]
[baud] USB baud rate              5 [1=9600,2=19200,3=38400,4=57600,5=115200,6=230400]
[gpl] default gcode plane         0 [0=G17,1=G18,2=G19]
[gun] default gcode units mode    1 [0=G20,1=G21]
[gco] default gcode coord system  1 [1-6 (G54-G59)]
[gpa] default gcode path control  2 [0=G61,1=G61.1,2=G64]
[gdi] default gcode distance mode 0 [0=G90,1=G91]
</pre>

Configuration is non-moded; that is, configuration lines and Gcode blocks can be used without changing modes. However, it is not recommended to intermingle configs with Gcode blocks, as changing configuration during a job may have unpredictable results. See the [Operating Model](g2dialect-Operating-Model) for details. Please note: in some future release configuration commands that arrive during a gcode cycle (i.e. job) may be rejected.


## Updating Settings 
To update a setting enter $token=value

Tokens are a mnemonic plus a group prefix (system settings have no prefix). The setting is taken and the value is echoed in a descriptive text line. No spaces are allowed. The following are examples of valid and invalid inputs. 

	Request | Response | Notes
	--------|----------|-------
	$yfr=800 | [yfr] y_feedrate_maximum        800.000 mm/min | Set Y axis feed rate maximum to 800 mm/min
	$yfr=16000 | [yfr] y_feedrate_maximum      16000.000 mm/min |  Set Y axis feed rate maximum to 16000 mm/min
	$2po=1 | [2po] m2_polarity                 1 [0,1] | Set motor 2 polarity
	$ex=1 | [ex]  enable_xon_xoff             1 [0,1] | Enable XON/XOFF protocol
	$ted=1 | error: Unrecognized command $ted | Example of an error