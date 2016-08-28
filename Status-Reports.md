## Status Reports
Status reports provide a real-time view of what's going on inside g2core by reporting the dynamic model. IOW they can tell you where you are (position), if the machine is moving, how fast, what it's doing, when a move is done, and other things as well. They can be returned on-demand, or can be automatically generated during movement so you can see move progress. They are delivered in JSON so the UI can use it, or can be delivered in plain text for a command-line user.  

The following variables can be reported in a status report

	Request | Response | Description
	---------|--------------|-------------
	stat | machine_state      | See [here](#stat-values)
	n | model line_number | Gcode line number `N` currently being read
	line | runtime_line_number | Runtime line number currently being executed
	vel | velocity | actual velocity - may be different than programmed feed rate 
	feed | feed_rate          | gcode programmed feed rate (F word) 
	unit | units_mode         | 0=inch, 1=mm
	coor | coordinate_system  | 0=g53, 1=g54, 2=g55, 3=g56, 4=g57, 5=g58, 6=g59
	momo | motion_mode        | 0=traverse, 1=straight feed, 2=cw arc, 3=ccw arc
	plan | plane_select       | 0=XY plane, 1=XZ plane, 2=YZ plane
	path | path_control_mode  | 0=exact stop, 1=exact path, 2=continuous
	dist | distance_mode      | 0=absolute distance mode, 1=incremental distance mode
	frmo | feed_rate_mode     | 0=units-per-minute-mode, 1=inverse-time-mode
	macs | machine state | internal state for deeper inspection
	cycs | cycle state | internal state for deeper inspection
	mots | motion state | internal state for deeper inspection
	hold | hold state | internal state for deeper inspection

The following are available for all axes, XYZABC. Only *posX* is shown for all axes for illustration. The rest are implied.

	Request | Response | Description
	---------|--------------|-------------
	posx | x work position | X work position in prevailing units (mm or inch) 
	_posy_ | _y work position_
	_posz_ | _z work position_
	_posa_ | _a work position_
	_posb_ | _b work position_
	_posc_ | _c work position_
	mpox | x absolute position | X machine position in absolute coordinate system (mm or inch).
	g92x | offset | G92 origin offset for X axis
	g54x | coord 1 offset | X axis G54 coordinate system offset
	g55x | coord 2 offset | X axis G55 coordinate system offset
	g56x | coord 3 offset | X axis G56 coordinate system offset
	g57x | coord 4 offset | X axis G57 coordinate system offset
	g58x | coord 5 offset | X axis G58 coordinate system offset
	g59x | coord 6 offset | X axis G59 coordinate system offset
	g28x | G28 position | X axis G28 saved position
	g30x | G30 position | X axis G30 saved position

It's worth noting that any gettable variable can be put in a status report - the above variables are listed as they represent the Gcode model state. For example `fv` would return the firmware version, but this would not change.

It's also worth noting that any variable can be independently queried as an individual variable, and axis variables (e.g. pos, G55) can be queried as a group, e.g. `pos`, or `g55`

## Text Mode Status Reports
### Text Mode On-Demand Status Reports
Enter a `?` to get a status report in text mode.

### Text Mode Automatic Status Reports
Automatically generated status reports may enabled with `$sv=1` or `$sv=2` and a corresponding status interval `$si=250` set in milliseconds. Reports will be generated for each new command entered, during movement every N milliseconds, and when the machine has stopped (i.e. at the end of the final move in the buffer).

Filtered status reports `$sv=1` return only variables that have changed since the previous report. This saves serial bandwidth and host processing time. We recommend using filtered reports.
<pre>
$sv=0      turn off automatic status reports
$sv=1      turn on filtered automatic status reports
$sv=2      turn on unfiltered automatic status reports
$si=250    request status reports every 250 ms during movement
</pre> 

## JSON Mode Status Reports
JSON mode status reports are parent/child objects with a "sr" parent and one or more child NV pairs, as in the example below.<br> 

<pre>
{"sr":{"line":1245,"posx":23.4352,"posx":-9.4386,"posx":0.125,"vel":600,"unit":"1","stat":"5"}}
</pre>

### JSON Mode On-Demand Status Reports
The following will return a single status report. The forms below are equivalent.
<pre>
{"sr":null}    request status report in strict JSON mode
{"sr":n}       same as above but null is abbreviated to n
{sr:n}         request status report in relaxed JSON mode

Note that JSON can be received in either strict or relaxed mode, but results 
will be will be returned in strict or relaxed mode depending on {js:N} setting
</pre> 

### JSON Mode Automatic Status Reports
Automatically generated status reports may enabled with `{sv:1}` or `{sv:2}`. Reports will be generated for each new command entered, during movement every N milliseconds according to `{si:N}`, and when the machine has stopped (i.e. at the end of the final move in the buffer).

Filtered status reports `{sv:1}` return only variables that have changed since the previous report. This saves serial bandwidth and host processing time. However, "stat" is always returned for a final report (i.e. after movement has stopped) so that UI's can reliably know that motion is complete (stat=3) or an M2/M30 program end has been received (stat=4). We recommend using filtered status reports for all UIs.
<pre>
{sv:0}        turn off automatic status reports
{sv:1}        turn on filtered automatic status reports
{sv:2}        turn on unfiltered automatic status reports
{si:250}      request status reports every 250 ms during movement
</pre> 

##Setting Status Report Fields
#### Set Status Report Fields - Firmware Build 440.21 and earlier
In addition to on-demand and automatic status reports, JSON mode also supports setting the fields to be displayed in status reports. Although this command is only supported in JSON mode, the fields set in JSON apply to both JSON and text mode reports.

Variables to be included in a status reports are selected by setting values to 'true'. These variables will be returned on subsequent SR requests in the order they were provided in the SET command. For example, the string below could be used to set up the status report in the example above. It will eliminate any previously recorded settings. Note that the `t` is not in quotes - it is actually an abbreviated JSON value for `true`, not a string that says "true". Example:
<pre>
{sr:{line:t,posx:t,posy:t,posz:t,vel:t,unit:t,stat:t}}
</pre> 

In firmware build 440.21 and below there is no incremental setting of variables - all variables are reset and must be specified in a single SET command. _See Incremental Status Report Setting (below) for further details_

#### Set Status Report Fields - Firmware Build 440.22 and later
Incremental status report setup is supported as of build 440.22. Everything above still applies except that sending a status report setup string will add to the existing settings. Behaviors are:

 * `{sr:f}` removes all status reports (clears)
 * `{sr:t}` restores status reports to default settings for the stored profile
 * `{sr:{<key1>:t,...<keyN>:t}}` adds key1 through keyN to the status report list
 * `{sr:{<key1>:f,...<keyN>:f}}` removes key1 through keyN from the status report list
 

  - Lines may have a mix of t and f pairs
  - List ordering is not guaranteed in the case of mixed removes and adds in the same command
 
Error conditions:
  - All failures leave original status report list untouched
  - A key that is not recognized fails with STAT_UNRECOGNIZED_NAME (stat=100)
  - A value other than 't', or 'f' fails with STAT_INPUT_VALUE_RANGE_ERROR (stat=110)
  - An attempt to add an element that exceeds list max fails with STAT_INPUT_EXCEEDS_MAX_LENGTH (stat=107)
  - Malformed JSON (bad syntax) fails as usual (stat=111)

## `stat`
`stat` is the **machine state** variable that can be queried directly or set to return in a status report.
```
    {stat:n}  queries this value
    
    {sr:"line", "posx", "posy", "posz", "feed", "vel", "momo", "stat"}

```
 
## Stat Values
```
typedef enum {
    COMBINED_INITIALIZING = 0,  // [0] machine is initializing
    COMBINED_READY,             // [1] machine is ready for use
    COMBINED_ALARM,             // [2] machine in alarm state
    COMBINED_PROGRAM_STOP,      // [3] program stop/no more blocks
    COMBINED_PROGRAM_END,       // [4] program end
    COMBINED_RUN,               // [5] machine is running
    COMBINED_HOLD,              // [6] machine is holding
    COMBINED_PROBE,             // [7] probe cycle active
    COMBINED_CYCLE,             // [8] reserved for canned cycles < not used >
    COMBINED_HOMING,            // [9] homing cycle active
    COMBINED_JOG,               // [10] jogging cycle active
    COMBINED_INTERLOCK,         // [11] machine in safety interlock hold
    COMBINED_SHUTDOWN,          // [12] machine in shutdown state
    COMBINED_PANIC              // [13] machine in panic state
} cmCombinedState;

	Number | Value | Description
	---------|--------------|-------------
	0 | initializing |Machine is initializing

