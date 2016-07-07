
# JSON Cheat Sheet
This table summarizes using JSON for [configuration and commands](https://github.com/synthetos/TinyG/wiki/TinyG-Configuration). Details are provided in the subsequent sections.

	Request | Response | Description
	---------|--------------|-------------
	{"xvm":n} | {"r":{"xvm":16000},"f":[1,0,11,1301]}<nl>| get X axis maximum velocity
	{"xvm":15000} | {"r":{"xvm":15000},"f":[1,0,14,9253]}<nl>| set X axis maximum velocity to 15000
	{"x":{"vm":n}} | {"r":{"x":{"vm":16000}},"f":[1,0,16,2128]}<nl>| alternate form to get X axis maximum velocity
	{"x":{"vm":15000}} | {"r":{"x":{"vm":15000}},"f":[1,0,19,2131]}<nl>| alternate form to set X axis maximum velocity to 15000
	{"x":n} | {"r":{"x":{"am":1,"vm":16000.000,"fr":16000.000,.... | get entire X axis group (see below for entire response)
	{"gc":"g0x10"} | {"f":[1,0,11,1234]}<nl>| send Gcode with verbosity=1, 2 or 3
	{"gc":"n20g0x20"} | {"r":{"n":20},"f":[1,0,9,5362]} | send Gcode with verbosity=4
	{"gc":"n20g0x20"} | {"r":{"gc":"n20g0x20","n":20},"f":[1,0,9,7209]} | send Gcode with verbosity=5
	{"gc":"g0x10"} | {"r":{"gc":"g0x10"},"f":[1,0,6,8628]}<nl>| send Gcode with verbosity=5
	n20g0x20 | {"r":{"gc":"n20g0x20","n":20},"f":[1,0,9,7209]} | send unwrapped Gcode with verbosity=5
	g0x10 | {"r":{"gc":"g0x10"},"f":[1,0,6,8628]}<nl>| send unwrapped Gcode with verbosity=5


X axis group response:
<pre>
{"r":{"x":{"am":1,"vm":40000,"fr":40000,"tn":0,"tm":420,"jm":5000,"jh":20000,"jd":null,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":4,"zb":2}},"f":[1,0,6]}</pre>