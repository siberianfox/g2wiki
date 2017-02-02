_Note - some of the information on this page will only be valid after Issue #237 is committed to edge, presumably as firmware build 100.20. See [Changes to Probing](#changes-to-probing-prior-to-10020) end end of this page for details._

This page is similar to, but not identical to:

- [LinuxCNC Gcode reference](http://linuxcnc.org/docs/devel/html/gcode/g-code.html)

##G38.x Probes
g2Core implements probing using standard Gcodes G38.2, G38.3, G38.4 and G38.5. In addition, probing settings and results are accessible using JSON commands and queries.

To run a probe the tool in the spindle (toolhead) must be a probe, contact a probe switch, or otherwise provide a reliable contact signal to the designated [probe input](#configuring-probe-input) _(Note: We don't recommend probing with tools, but some people do this. Tool coatings can be a problem.)_

To initiate a probe cycle send `G38.x AXES Fnnn`. Any axis or axes may be used (XYZABC). The axis words together define the target point that the probe will move towards, starting from the current position. Axis words are optional but at least one of them must be provided. Feed rate must be non-zero, set either in the Gcode block or previously set (F is modal). 

In a probing cycle the toolhead moves towards the target in a straight line at the defined feed rate. _In inverse time feed mode, the feed rate is such that the whole motion from the current position to the target would take the specified time._ The move stops (within machine acceleration limits) when the target is reached or when the requested change in the probe input takes place, whichever occurs first. The probe may then perform a small backoff move to position the tool at the exact motor step on which the probe input tripped. This will be seen as a second move. 

After probing is complete results are available in a JSON probe report. The probe report contains probe status `e` set to 1 for success, or 0 if not tripped, and the end positions of the axes. Positions are in machine coordinates (absolute coordinates) and in millimeters. A probe report can be invoked by requesting requesting `{prb:n}`. Values may also be queried individually as `{prbe:n}`, `{prbz:n}` or similar.

A probe report can also be automatically returned if `{prbr:t}` has been set (default value). Set `{prbr:f}` to disable automatic reports.
 
Examples:

```
{"prb":n}  --> {"r":{"prb":{"e":1,"x":0,"y":0,"z":-8.804,"a":0,"b":0,"c":0}},"f":[1,0,9]}
{"prbe":n} --> {"r":{"prbe":1},"f":[1,0,10]}
{"prbz":n} --> {"r":{"prbz":-8.804},"f":[1,0,10]}
{"prbr":true}  enable automatic probe report
{"prbr":1}     enable automatic probe report (also works)
{prbr:t}       enable automatic probe report (shorter form)
```

To query end positions in work coordinates use `{pos:n}` which will report using current offsets and units mode.

A "failed" probe is where the probe is not tripped before the target is reached. A failed G38.2 or G38.4 probe will put the machine in an [Alarm](Alarm-Processing) state, preventing further actions from occurring until the alarm is cleared using any of: `{clear:n}`, `{clr:n}`, or `$clear`. Conversely, a failed G38.3 or G38.5 probe will not trigger an alarm. In either case, probe status `{prbe:n}` is set to 0, indicating that the probe was unsuccessful. Also note that other exceptions - such as those listed in the Error list, below, will Alarm regardless of the probe type. 

Spindle and coolant are not affected by probing, and will remain on or off during a probing operation.

Also note that soft limits are disabled during a probe cycle, allowing probes to be performed to the edge of the work volume by extending slightly beyond the work volume. _This behavior may be refined in the future, but for now it should address some of the probe failures users have been experiencing_

It is an error if:

- The current position is the same as the target point
- No axis word is provided
- Feed rate is zero
- No input is [configured for probing](#configuring-probe-input)
- The probe input is already in the 'tripped' state

##Probing Command Reference

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G38.2 | _axes_ Fnnn | probe toward workpiece | stop on contact (switch closed), alarm if probe does not trip
	G38.3 | _axes_ Fnnn | probe toward workpiece | stop on contact (switch closed)
	G38.4 | _axes_ Fnnn | probe away from workpiece | stop on loss of contact (switch open), alarm if probe does not trip
	G38.5 | _axes_ Fnnn | probe away from workpiece | stop on loss of contact (switch open)
	{prb:n} | none | probe results | Probe status `e` and axis positions after probe, in absolute machine coordinates, in MM. Read only
	{prbe:n} | none | probe status | 0=failed, 1=succeeded. Read only
	{prbz:n} | none | probe result Z | Z axis position, as above. Other axes are similar
	{prbr:_} | t/f | enable probe reports | If true, generate automatic probe reports when probing is complete


### Configuring Probe Input
The digital input to be used by probing is selected by setting the input function to INPUT_FUNCTION_PROBE (4). In most cases this is the Zmin input, but does not have to be. An example of how to assign the probe input to digital input #5 is shown below. 
```
{di5:n}   --> {"r":{"di5":{"mo":0,"ac":0,"fn":0}},"f":[1,0,9]}   Query DI5 before setting function
{di5fn:4} --> {"r":{"di5fn":4},"f":[1,0,11]}                     Set function to 4 (see below)
{di5:n}   --> {"r":{"di5":{"mo":0,"ac":0,"fn":4}},"f":[1,0,9]}   Query DI5 after setting function
```
The input functions are:
```
INPUT_FUNCTION_NONE = 0,
INPUT_FUNCTION_LIMIT = 1,           // limit switch processing
INPUT_FUNCTION_INTERLOCK = 2,       // interlock processing
INPUT_FUNCTION_SHUTDOWN = 3,        // shutdown in support of external emergency stop
INPUT_FUNCTION_PROBE = 4,           // assign input as probe input
```
The Zmin input (typically input 5) is assigned by default unless specified in the settings of overridden in the settings file or by a JSON command. If the input is changed from Zmin, then Zmin must be de-assigned manually. If more than one input is configured for probing the lowest numbered input is used. This is not returned as an error. Use {di:n} to see all digital inputs configured.

Other digital input settings are: 

- `mo` - set input mode to 0 for active-low (normally-open) operation and 1 for active-hi (normally-closed) operation (NB: at some time in the future these values will change). Most contact probes are active-low, so this value should be st to 0.
- `ac` - action on switch closure. Unused by probing. Set to 0.

See also: [Digital IO](Digital-IO).

## Using Probes for Tramming (Bed Leveling)
See [Tramming]()

## Changes To Probing prior to 100.20
The following changes were made to probing in firmware build 100.20

- The probe input is assignable, and must be configured. The Zmin input is assigned by default.
- Previously probes were not allowed to have ABC axes. Now they can.

