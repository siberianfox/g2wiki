Reserved for Probe commands. Note - some of the information on this page will only be valid Issue #237 is committed to edge. These cases are labeled.

This page is similar to, but not identical to:

- [LinuxCNC Gcode reference](http://linuxcnc.org/docs/devel/html/gcode/g-code.html)

##G38.x Probes
g2Core implements probing using the standard Gcodes G38.2, G38.3, G38.4 and G38.5. In addition, probing settings and results are accessible using JSON commands and queries.

To run a probe the tool in the spindle (toolhead) must be a probe, contact a probe switch, or otherwise provide a reliable contact signal to the designated [Probe Input]() (Note: We don't recommend probing with tools, but some people do this. Tool coatings can be a problem.)

To initiate a probe cycle send `G38.x AXES Fnnn` to perform a straight probe operation. Any axis or axes may be used (XYZABC). The axis words together define the target point that the probe will move towards, starting from the current position. Axis words are optional but at least one of them must be used. Feed rate must be non-zero, set either in the above Gcode block, or previously set (F is modal). 

In a probing cycle the toolhead moves towards the target in a straight line at the current feed rate. _In inverse time feed mode, the feed rate is such that the whole motion from the current position to the target would take the specified time._ The move stops (within machine acceleration limits) when the target is reached or when the requested change in the probe input takes place, whichever occurs first. The probe may then perform a small backoff move to position the tool at the exact motor step on which the probe input tripped. This will be seen as a second move. 

After successful probing the probe results are available by requesting `{prb:n}`. This contains probe status `e` set to 1 (true), and the the end position of all axes. Positions are in machine coordinates (absolute coordinates) and in millimeters. Values may also be queried individually as '{prbe:n}`, {prbz:n}` or similar. Examples:

```
{"prb":n}  --> {"r":{"prb":{"e":1,"x":0,"y":0,"z":-8.804,"a":0,"b":0,"c":0}},"f":[1,0,9]}
{"prbe":n} --> {"r":{"prbe":1},"f":[1,0,10]}
{"prbz":n} --> {"r":{"prbz":-8.804},"f":[1,0,10]}
```

To query end positions in work coordinates use `{pos:n}` which will report using current offsets and units mode.


If the probe is not tripped before the target is reached G38.2 and G38.4 will put the machine in an Alarm state, preventing further actions from occurring until the alarm is cleared using any of: `{clear:n}`, `{clr:n}`, or `$clear`. The probe status `{prbe:n}` will read 0 (false), indicating that the probe was unsuccessful.

If the probe is not tripped before the target is reached G38.3 and G38.5 the probe status `{prbe:n}` will read 0 (false), indicating that the probe was unsuccessful. No alarm is triggered.


parameters 5061 to 5069 will be set to the coordinates of X, Y, Z, A, B, C, U, V, W of the location of the controlled point at the time the probe changed state. After unsuccessful probing, they are set to the coordinates of the programmed point. Parameter 5070 is set to 1 if the probe succeeded and 0 if the probe failed. If the probing operation failed, G38.2 and G38.4 will signal an error by posting an message on screen if the selected GUI supports that. And by halting program execution.

A comment of the form (PROBEOPEN filename.txt) will open filename.txt and store the 9-number coordinate consisting of XYZABCUVW of each successful straight probe in it. The file must be closed with (PROBECLOSE). For more information see the Comments Section.

An example file smartprobe.ngc is included (in the examples directory) to demonstrate using probe moves to log to a file the coordinates of a part. The program smartprobe.ngc could be used with ngcgui with minimal changes.

It is an error if:

the current point is the same as the programmed point.

no axis word is used

cutter compensation is enabled

the feed rate is zero

the probe is already in the target state




##Gcode and JSON Command Reference

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G38.2 | F axes | probe toward workpiece | stop on contact (switch closed), alarm if failure
	G38.3 | F axes | probe toward workpiece | stop on contact (switch closed)
	G38.4 | F axes | probe away from workpiece | stop on loss of contact (switch open), alarm if failure
	G38.5 | F axes | probe away from workpiece | stop on loss of contact (switch open)
	{prbe:n} | none | probe status | 0=failed, 1=succeeded. Read only
	{prb:n} | none | probe result vector | Axis positions after probe, in absolute machine coordinates, in MM. Read only
	{prbz:n} | none | probe result Z | Z axis position, as above. Other axes are similar


- Operation of G38.2, 3, 4, 5

- Set  prob input INPUT_FUNCTION_PROBE
```
Zmin example
```

- small backoff
- prbe, 
- prbx ... a . Explain these are in MACHINE coords. If you want work coords see posx...a

## Using Probes for Tramming (Bed Leveling)

Changes
- No more automatic probe report, Use SRs or {prb:n} request

