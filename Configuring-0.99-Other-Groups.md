_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_



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


##PWM Group (Pulse Width Modulation)
There is currently only one PWM channel (p1), but the configs are structured for multiple PWM groups. The PWM channel is set up to act as a remote control Electronic Speed Controller (ESC), but can be used for other PWM functions using these settings. 

	Setting | Description | Notes
	--------|-------------|-------
	{p1frq:_} | Frequency | in Hz, e.g. 100
	{p1csl:_} | Clockwise speed low | In RPM - arbitrary units unless you calibrate it, e.g. 1000
	{p1csh:_} | Clockwise speed high | In RPM
	{p1cpl:_} | Clockwise phase low | 0.000 to 1.000, e.g. 0.125 for 12.5% phase angle
	{p1cph:_} | Clockwise_phase_high | 0.000 to 1.000
	{p1wsl:_} | CCW speed low | In RPM 
	{p1wsh:_} | CCW speed high | In RPM
	{p1wpl:_} | CCW phase low | 0.000 to 1.000
	{p1wph:_} | CCW phase high | 0.000 to 1.000
	{p1pof:_} | Phase off | 0.000 to 1.000 used to set OFF phase for PWM devices that are not off at 0 phase