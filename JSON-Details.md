This page describes JSON operations in G2. This page is really a continuation of [this page](JSON-Operation)

## JSON Mode Protocol
### Commands and Responses
Commands in JSON mode are sent as JSON objects. Some request and response examples:

    {xfr:n}         {"r":{"xfr":12000.000},"f":[3,0,6]}   get the X axis max feed rate
    {xfr:12000}     {"r":{"xfr":12000.000},"f":[3,0,6]}   set the X maximum feed rate to 12000 mm/min (assuming G21 mode)
    {x:n}           {"r":{"x":{"am":1,"vm":16000.000,.... get all configuration settings for the X axis
    {gc:"g0 x100"}  {"r":{"gc":"g0x100"},"f":[3,0,6]}     execute the Gcode block "G0 X100"

    where  "f":[3,0,6]  
       is  "f":[<protocol_version>, <status_code>, <available_buffer_count>]

Responses are wrapped in a response body and include a footer. Requests and responses are a single line of text terminated by CR, LF or CR/LF depending on the board configuration settings ($ec, $el). Requests and responses can be up to 256 characters long but are generally much shorter. <br>

The footer contains a 3 element array:

Element  | Notes
-------|-------------------------
array_ID/version | Initially set to 3 to indicate line mode communications. This number identifies the array and version for the parser. It will be incremented if protocol changes are made that would affect hosts/clients, or is 2 if in character mode communications
status_code | 0 means the command executed `OK`. All others are exceptions. See [Status Codes](Status-Codes) for details.
buffers_available | Indicates how many line buffers are available in the receive buffer pool. This allows the host to check if they are operating at the right level of queueing.

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
{xvm:n} | {"r":{"xvm":16000},"f":[3,0,6]}<nl>| get X axis maximum velocity
{x:{vm:n}} | {"r":{"x":{"vm":16000}},"f":[3,0,6]}<nl>| alternate form to get X axis maximum velocity
{x:n} | {"r":{"x":{"am":1,"vm":16000.000,"fr":16000.000,.... | get entire X axis group

### Setting Configuration Parameters (like a POST)
To set a parameter pass an object with the value to be set. The value applied is returned in the response. The response value may be different than the requested valued in some cases. For example, an attempt to set a status report interval less than the minimum will return the minimum interval. Trying to set a read-only value will return that value; for example, firmware version. In some other cases a value of `false` or `null` may be returned. The following are examples of valid set commands. All requests and responses are on a single line of text (even if the table below wraps on your display).

Request | Response | Description
---------|--------------|-------------
{xvm:15000} | {"r":{"xvm":15000},"f":[3,0,6]} | set X axis maximum velocity to 15000
{x:{vm:15000}} | {"r":{"x":{"vm":15000},"f":[3,0,6]}} | alternate form to set X axis maximum velocity to 15000
{si:250} | {"r":{"si":250},"f":[3,0,6]} | Set status minimum interval to 250 ms.
{si:10} | {"r":{"si":200},"f":[3,0,6]} | Try to set status interval to 10 ms, but minimum was 200
{fv:2.0} | {"r":{"fv":0.950},"f":[3,0,6]} | Version stays 0.95 despite your wishes :(

### Groups
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

Values within a group can be set by including that member in the set command. Examples:
<pre>
Set velocity max and feed rate max only (other values are unchanged)
{"x":{"vm":1500,"fr":1500}}

Set all values in X
{"x":{"am":1,"vm":1500,"fr":1500,"tm":150,"jm":600000000,"jh":400000000,"jd":0.0100,"sn":1,"sx":0,"sv":750,"lv":150,"lb":5.000,"zb":0.000}}
</pre>

**Diagnostic Groups**<br>
In addition the the operating groups above a series of diagnostic groups can be enabled at compile time See [Diagnostics](Diagnostics)

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
In JSON mode Gcode blocks may be provided either as native gcode text or wrapped in JSON as a "gc" object. In either case responses will be returned in JSON format. A JSON wrapper may only contain a single gcode block. The following are all valid Gcode blocks that can be sent to G2<br>

Gcode Request | Response | Notes
---------|--------------|---------
{gc:"g0 x100"} | {"r":{"gc":"G0X100"},"f":[3,0,6]} | wrapped gcode, [strict syntax](JSON-Operation#json-syntax-option---relaxed-or-strict)
{gc:"g0 x100"} | {r:{gc:"G0X100"},f:[3,0,6]} | wrapped gcode, [relaxed syntax](JSON-Operation#json-syntax-option---relaxed-or-strict)
g0 x100 | {"r":{"gc":"G0X100"},"f":[3,0,6]} | unwrapped gcode
{gc:"g0 x100 (Initial move)"} | {"r":{"gc":"G0X100 (Initial move)"},"f":[3,0,6]} |Gcode with comment
{gc:"m6 t2 (msgChange tool)"} | {"r":{"gc":"M6T2","msg":"Change tool"},"f":[3,0,6]} | Gcode with message in comment

The first responses are pretty normal. The fourth has a comment in it. The fifth is what would happen if a MSG were communicated in the Gcode comment.
