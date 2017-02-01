_Note - some of the information on this page will only be valid after Issue #237 is committed to edge, presumably as firmware build 100.20. See [Changes to Probing](#changes-to-probing-prior-to-10020) for details._

This page is similar to, but not identical to:

- [LinuxCNC Gcode reference](http://linuxcnc.org/docs/devel/html/gcode/g-code.html)

##G38.x Probes
g2Core implements probing using standard Gcodes G38.2, G38.3, G38.4 and G38.5. In addition, probing settings and results are accessible using JSON commands and queries.

To run a probe the tool in the spindle (toolhead) must be a probe, contact a probe switch, or otherwise provide a reliable contact signal to the designated [probe input](#configuring-probe-input) _(Note: We don't recommend probing with tools, but some people do this. Tool coatings can be a problem.)_

To initiate a probe cycle send `G38.x AXES Fnnn`. Any axis or axes may be used (XYZABC). The axis words together define the target point that the probe will move towards, starting from the current position. Axis words are optional but at least one of them must be used. Feed rate must be non-zero, set either in the above Gcode block, or previously set (F is modal). 

In a probing cycle the toolhead moves towards the target in a straight line at the current feed rate. _In inverse time feed mode, the feed rate is such that the whole motion from the current position to the target would take the specified time._ The move stops (within machine acceleration limits) when the target is reached or when the requested change in the probe input takes place, whichever occurs first. The probe may then perform a small backoff move to position the tool at the exact motor step on which the probe input tripped. This will be seen as a second move. 

After a probe, results are available by requesting `{prb:n}`. This contains probe status `e` set to 1 for success, or 0 if not tripped, and it lists the end position of all axes. Positions are in machine coordinates (absolute coordinates) and in millimeters. Values may also be queried individually as '{prbe:n}`, {prbz:n}` or similar. Examples:

```
{"prb":n}  --> {"r":{"prb":{"e":1,"x":0,"y":0,"z":-8.804,"a":0,"b":0,"c":0}},"f":[1,0,9]}
{"prbe":n} --> {"r":{"prbe":1},"f":[1,0,10]}
{"prbz":n} --> {"r":{"prbz":-8.804},"f":[1,0,10]}
```

To query end positions in work coordinates use `{pos:n}` which will report using current offsets and units mode.

If the probe is not tripped before the target is reached G38.2 and G38.4 will put the machine in an Alarm state, preventing further actions from occurring until the alarm is cleared using any of: `{clear:n}`, `{clr:n}`, or `$clear`. Conversely, G38.3 and G38.5 will not trigger an alarm. In either case, probe status `{prbe:n}` is set to 0, indicating that the probe was unsuccessful.

It is an error if:

- The current position is the same as the target point
- No axis word is provided
- Feed rate is zero
- The probe input is already in the 'tripped' state

##Gcode and JSON Command Reference

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G38.2 | F axes | probe toward workpiece | stop on contact (switch closed), alarm if failure
	G38.3 | F axes | probe toward workpiece | stop on contact (switch closed)
	G38.4 | F axes | probe away from workpiece | stop on loss of contact (switch open), alarm if failure
	G38.5 | F axes | probe away from workpiece | stop on loss of contact (switch open)
	{prb:n} | none | probe result vector | Probe status and axis positions after probe, in absolute machine coordinates, in MM. Read only
	{prbe:n} | none | probe status | 0=failed, 1=succeeded. Read only
	{prbz:n} | none | probe result Z | Z axis position, as above. Other axes are similar


### Configuring Probe Input
The digital input to be used by probing is selected by setting the input function to INPUT_FUNCTION_PROBE. In most cases this is the Zmin input, but does not have to be. An example of how to assign the probe input to digital input #5 is shown below. See also: [Digital IO](Digital-IO)
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
Other settings are: 

- `mo` - set to 0 for active-low (normally-open) operation and 1 for active-hi (normally-closed) operation (NB: at some time in the future these values will change)
- `ac` - action on switch closure. Unused for probing. Set to 0.

If more than one input is configured for probing the lowest numbered input is used. This is not returned as an error. Use {di:n} to see all digital inputs configured.

## Using Probes for Tramming (Bed Leveling)
See [Tramming]()

## Changes To Probing prior to 100.20
The following changes were made to probing in firmware build 100.20

- The probe input is assignable, and must be configured. Previously it was hard wired to Zmin or Zmax
- Probes no longer generate an automatic probe report, Use SRs or {prb:n} request
- Previously probes were not allowed to have ABC axes. Now they can

