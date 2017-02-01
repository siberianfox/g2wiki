Reserved for Probe commands. Note - some of the information on this page will only be valid Issue #237 is committed to edge. These cases are labeled.

This page is based on, but not identical to, the following references: 

- [LinuxCNC Gcode reference](http://linuxcnc.org/docs/devel/html/gcode/g-code.html)
- [Tormach G43 Page](http://www.tormach.com/g43_g44_g49.html)

##G38.x Probe

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	G38.2 | F <axes> | probe toward workpiece | stop on contact, signal error if failure
	G38.3 | F <axes> | probe toward workpiece | stop on contact
	G38.4 | F <axes> | probe away from workpiece | stop on loss of contact, signal error if failure
	G38.5 | F <axes> | probe away from workpiece | stop on loss of contact


- Operation of G38.2, 3, 4, 5

- Set  prob input INPUT_FUNCTION_PROBE
```
Zmin example
```

- small backoff

- prbe, 



- prbx ... a . Explain these are in MACHINE coords. If you want work coords see posx...a

Changes
- No more automatic probe report, Use SRs or {prb:n} request

