This page describes JSON operations in G2. This page is really a continuation of [this page](JSON-Operation)

##JSON Mode Protocol
### Commands and Responses
Commands in JSON mode are sent as JSON objects. Some request and response examples:

    {"xfr":n}        {"r":{"xfr":12000.000},"f":[1,0,14,3009]}   get the X axis max feed rate
    {"xfr":12000}     {"r":{"xfr":12000.000},"f":[1,0,14,3009]}   set the X maximum feed rate to 12000 mm/min (assuming the system is in G21 mode)
    {"x":n}          {"r":{"x":{"am":1,"vm":16000.000,....       get all configuration settings for the X axis
    {"gc":"g0 x100"}  {"r":{"gc":"g0x100"},"f":[1,0,17,9360]}     execute the Gcode block "G0 X100"

    where  "f":[1,0,255,1234]  
       is  "f":[<protocol_version>, <status_code>, <input_available>, <checksum>]

Responses are wrapped in a response body and include a footer. Requests and responses are a single line of text terminated by CR, LF or CR/LF depending on the board configuration settings ($ec, $el). Requests and responses can be up to 256 characters long but are generally much shorter. <br>

The footer contains a 4 element array:

	Element  | Notes
	-------|-------------------------
	array_ID/version | Initially set to 1. This number identifies the array and version for the parser. It will be incremented if protocol changes are made that would affect hosts/clients
	status_code | 0 means the command executed `OK`. All others are exceptions. See [Status Codes](https://github.com/synthetos/TinyG/wiki/TinyG-Status-Codes) for details.
	RX_received | Indicates how many characters were removed from the serial RX buffer to process this command. This allows the host to keep a running total of the bytes available in the TinyG RX buffer. The RX buffer starts with 254 bytes free. The terminating `<LF>` or `<CR>` counts as a byte.
	checksum | A hashcode checksum for the line; typically 4 digits but may be anything from 1 to 4 digits. The checksum is generated for the JSON line up to but not including the comma preceding the checksum itself. I.e, the comma is where the nul termination would exist. The checksum is computed as a [Java hashcode](http://en.wikipedia.org/wiki/Java_hashCode(\)) from which a modulo 9999 is taken to limit the length to no more than 4 characters. See compute_checksum() in util.c for C code.

### Response Verbosity
Character echo ($ee) is always an option; it's just not a good one for JSON. In JSON mode it should be turned off (  $ee=0, or {"ee":0}  ). It's a matter of preference for text mode.

JSON verbosity ($jv) sets the level of verbosity in JSON responses. It is set by {"jv":N} where N is one of:

	Num | label | description
	----|-------|-------------
	0 | JV_SILENT | No response is provided for any command
	1 | JV_FOOTER | Returns footer only - no command echo, gcode blocks or messages
	2 | JV_MESSAGES | Returns footer, messages (exception messages and gcode comment messages)
	3 | JV_CONFIGS | Returns footer, messages, config commands
	4 | JV_LINENUM | Returns footer, messages, config commands, gcode line numbers if present
	5 | JV_VERBOSE | Returns footer, messages, config commands, gcode blocks

Generally levels 2, 3 or 4 are recommended.

### Reading Configuration Parameters (like a GET)
To get a parameter pass an object with a null value. The value is returned in the response. Some examples of valid requests and responses are provided below. 

	Request | Response | Description
	---------|--------------|-------------
	{"xvm":n} | {"r":{"xvm":16000},"f":[1,0,11,1301]}<nl>| get X axis maximum velocity
	{"x":{"vm":n}} | {"r":{"x":{"vm":16000}},"f":[1,0,16,2128]}<nl>| alternate form to get X axis maximum velocity
	{"x":n} | {"r":{"x":{"am":1,"vm":16000.000,"fr":16000.000,.... | get entire X axis group

###Setting Configuration Parameters (like a POST)
To set a parameter pass an object with the value to be set. The value applied is returned in the response. The response value may be different than the requested valued in some cases. For example, an attempt to set a status report interval less than the minimum will return the minimum interval. Trying to set a read-only value will return that value; for example, firmware version. In some other cases a value of 'false' will be returned. The following are examples of valid set commands. All requests and responses are on a single line of text (even if the table below wraps on your display).

	Request | Response | Description
	---------|--------------|-------------
	{"xvm":15000} | {"r":{"xvm":15000},"f":[1,0,14,9253]} | set X axis maximum velocity to 15000
	{"x":{"vm":15000}} | {"r":{"x":{"vm":15000},"f":[1,0,19,2131]}} | alternate form to set X axis maximum velocity to 15000
	{"si":250} | {"r":{"si":250.000},"f":[1,0,19,2131]} | Set status minimum interval to 250 ms. 
	{"si":10} | {"r":{"si":50.000},"f":[1,0,19,2131]} | Try to set status interval to 10 ms, but minimum was 50
	{"fv":2.0} | {"r":{"fv":0.950},"f":[1,0,19,2131]} | The version stays at 0.95 despite your wishes :(

### Groups
The following groups can be read and set using JSON. The JSON strings provided below (i.e. the group with a null value ("")) will return all members of the group. 
<pre>
{"1":n}    return motor 1 settings
{"2":n}
{"3":n}
{"4":n}
{"x":n}    return X axis settings
{"y":n}
{"z":n}
{"a":n}
{"b":n}
{"c":n}
{"sys":n}  return system settings
{"pos":n}  return work coordinate positions fox XYZABC axes. In mm or inches depending on G20/G21
{"mpo":n}  return absolute machine positions fox XYZABC axes. Always in mm, regardless of G20/G21
{"ofs":n}  return current offsets fox XYZABC axes. Sums coordinate system and G92 offsets. in mm.
{"hom":n}  return homing state fox XYZABC axes, and 'e' for the entire machine. 1=homed, 0=not.
{"p1":n}   return PWM channel 1 settings (currently there is only 1 PWM channel)
{"g54":n}  return offsets for work coordinate system #1 (G54)
{"g55":n}  #2
{"g56":n}  #3
{"g57":n}  #4
{"g58":n}  #5
{"g59":n}  #6
{"g92":n}  return G92 offsets currently in effect
{"g28":n}  return coordinate saved by G28 command
{"g30":n}  return coordinate saved by G30 command
</pre>

Values within a group can be set by including that member in the set command. Examples:
<pre>
Set velocity max and feed rate max only (other values are unchanged)
{"x":{"vm":1500,"fr":1500}}

Set all values in X
{"x":{"am":1,"vm":1500,"fr":1500,"tm":150,"jm":600000000,"jh":400000000,"jd":0.0100,"sn":1,"sx":0,"sv":750,"lv":150,"lb":5.000,"zb":0.000}}
</pre>

### Status messages
Status messages are end-user text associated with each status code, similar to those on the [Status Codes](https://github.com/synthetos/TinyG/wiki/TinyG-Status-Codes) page. JSON does not return these whereas text mode does. JSON parsers must therefore work to the status codes, and provide their own end-user messages.

**HOWEVER**<br>
There are 2 cases where messages are returned in responses

* System generated advisory messages may be returned as "msg" tags for some advisory cases. For example, TinyG will accept a microstep value that the chips don't support so that people who wire up TinyG to run external steppers can still set non-standard microstep values.

* Gcode provided messages. A Gcode block can have a comment that starts with the string "msg". For example, (msgChange tool and hit cycle start) will return "Change tool and hit cycle start" as data for a "msg" tag. Presumably the UI would want to display this message.

### Gcode line numbers 
Gcode line numbers are returned as `n` objects if they are provided in a Gcode file - i.e. the line starts with Nxxxx where xxxxx is a number. An example is:

	Request | Response | Description
	---------|--------------|-------------
	N20 G1 F240 X2.01 Y2.99 | "n":20 | is returned as part of the response in the response

## Gcode in JSON Mode
In JSON mode Gcode blocks may be provided either as native gcode text or wrapped in JSON as a "gc" object. In either case responses will be returned in JSON format. A JSON wrapper may only contain a single gcode block.<br> 

	Gcode Request | Response | Notes
	---------|--------------|---------
	{"gc":"g0 x100"} | {"r":{"gc":"G0X100"},"f":[1,0,19,2131]} | wrapped gcode, [strict syntax](JSON-Operation#json-syntax-option---relaxed-or-strict)
	{gc:"g0 x100"} | {r:{gc:"G0X100"},f:[1,0,19,2131]} | wrapped gcode, [relaxed syntax](JSON-Operation#json-syntax-option---relaxed-or-strict)
	g0 x100 | {"r":{"gc":"G0X100"},"f":[1,0,19,2131]} | unwrapped gcode
	{"gc":"g0 x100 (Initial move)"} | {"r":{"gc":"G0X100 (Initial move)"},"f":[1,0,19,2131]} |Gcode with comment
	{"gc":"m0 (msgChange tool)"} | {"r":{"gc":"M0","msg":"Change tool"},"f":[1,0,19,2131]} | Gcode with message in comment

The first responses are pretty normal. The third has a comment in it. The fourth is what would happen if a MSG were communicated in the Gcode comment. 