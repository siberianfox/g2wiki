This page is the start page describing JSON operation in G2. This page provides a JSON cheat sheet and an overview of JSON operation.

Related Pages:
- [JSON Details](JSON-Details)
- [JSON Active Comments](JSON-Active-Comments)

# JSON Cheat Sheet
This table summarizes using JSON for [configuration and commands](Configuration-for-Firmware-Version-0.98). Details are provided in the subsequent sections.

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
 {"r":{"x":{"am":1,"vm":16000.000,"fr":16000.000,"tm":220.000,"jm":5000000000.000,"jd":0.010,"sn":3,"sx":2,"sv":3000.000,"lv":100.000,"lb":20.000,"zb":3.000}},"f":[1,0,9,9580]} 
</pre>
#JSON Summary
## General

TinyG can accept commands in [JSON](http://json.org/) (JavaScript Object Notation) mode or text mode. Using JSON simplifies writing host GUIs in languages such as Python, Ruby, JavaScript, Java, Processing and other languages that support dictionaries / hashmaps. 

Most commands that are available in JSON are also available in text mode, but there are some differences. ASCII communications overhead is somewhat higher in JSON than text mode but is still quite efficient and manageable.

####Syntax Notes
- The examples above are in strict JSON mode, as specified by json.org. TinyG will also accept input in relaxed JSON mode which omits the quotes on all fields except if the value field is a string.
- The following values can be used in full or by their abbreviations:
  - 'null' or 'n'
  - 'true' or 't'
  - 'false' or f 

### Is this RESTful?
The JSON interface is modeled as a RESTful interface, albeit running over USB serial and not HTTP. Using JSON enables exposing system internals as [RESTful resources](http://en.wikipedia.org/wiki/Representational_state_transfer), which makes the embedded system much easier to manipulate using modern programming techniques. Data is treated as resources and state is transfered in and out. Unlike RESTful HTTP, methods are implied by convention as there is no request header in which to declare them. 

## Basic Concepts
In JSON mode TinyG expects well structured JSON (if in doubt use the [JSON validator](http://jsonlint.com)). JSON requests are generally just curly braces and formatting around what would otherwise be a command line request. A JSON line can only contain a single request object. This may be a single value such as {"xvm":null}, or a complete group such as {"x":null} - but only one group. Note that groups with multiple *elements* are accepted. For example, this is OK: 
{"x":{"vm":16000,"fr":10000}}

### How things are encoded in JSON

	Term | Description
	---------------|--------------
	**name** | A name is a JSON name (aka **token**) describing a single data value or a group of data values. Examples of names include "xfr" referring to the X axis maximum feed rate, or "x" referring to all values associated with the X axis (the X axis group).<br>Names are not case sensitive.
	**value** | A value is a number, a quoted string, true/false, or null (as per JSON spec).<br>True and false values can be represented as `true` and `false` or `t` and `f` for short<br>NULL values can be represented by the word `null` (case insensitive), or simply `n` for short<br>Null values signal a GET, all others will set (PUT) the value, or in some cases invoke an action.
	**NVpair** | A name and a value is a name:value pair or NV pair
	**group** | A group is a collection of one or more NV pairs. Groups are used to specify all parameters for a motor, an axis, a PWM channel, or other logical grouping. A group is similar in concept to a RESTful resource or composite.

### What is encoded in JSON

	Term | Description
	---------------|--------------
	**config** | A **config** is a static configuration setting for some aspect of the machine. These parameters are not changed by Gcode execution (but see the G10 exception). Xfr is an example of a config. So is 1po. So is the X group.
	**block** | **Gcode blocks** are lines of gcode consisting of one or more gcode words, optional comments and possibly gcode messages
	**word** | **Gcode words** encode gcode commands. G1 is an example of a gcode word. So is x23.43. [Gcode supported by TinyG is listed here.](https://github.com/synthetos/TinyG/wiki/TinyG-Gcode-Support)  
	**comment** | A **Gcode comment** is denoted by parentheses - (this is a gcode comment). 
	**message** | A **Gcode message** is a special form of comment that is echoed to the machine operator. It's the part of the comment that follows a `(msg` preamble. For example: (msgThis part is echoed to the user). 

## JSON Overview & TinyG Subset

The concise JSON language definition is [here](http://json.org). A handy validator can be found [here](http://jsonlint.com).

TinyG implements a subset of JSON with the following limitations: 

* Supports 7 bit ASCII characters only 
* Supports decimal numbers only (no hexadecimal numbers or other non-decimals)
* Arrays are returned but are not (yet) accepted as input
* Names (tokens) are case-insensitive and cannot be more than 5 characters
* Groups cannot contain more than 24 elements (name/value pairs)
* JSON input objects cannot exceed 254 characters in total length. Outputs can be up to 512 chars.
* Limited object nesting is supported (you won't see more than 3 levels)
* All JSON input and output is on a single text line. There is only one `<LF>`, it's at the end of the line (broken lines are not supported)

##JSON Request and Response Formats
JSON requests are used to perform the following actions {with examples}

* Return the value of a single setting or state variable {"1mi":n}
* Return the values of a group of settings or state variables (aka a Resource) {"1":n}
* Set a single setting or state variable (note that most state variables are read-only) {"1mi":8}
* Set a multiple settings or state variables in a group {"1":{"po":1,"mi":8}}
* Submit a block (line) of Gcode to perform any supported Gcode command {"gc":"n20g1f350 x23.4 y43.2"}
* Special functions and actions;
 * Request a status report {"sr":n} 
 * Set status report contents {"sr":{"line":true,"posx":true,posy":true,   ...}}
 * Run self tests {"test":1}
 * Reset parameters to defaults {"defa":true}

JSON responses to commands are in the following general form.
<pre>
{"xjm":n} returns:
{"r":{"xjm":5000000000.000},"f":[1,0,11,6649]}

{"2":n} returns:
{"r":{"2":{"ma":1,"sa":1.800,"tr":36.540,"mi":8,"po":1,"pm":1}},"f":[1,0,9,2423]}
</pre>

The 'r' is the response envelope. The body of the response is the result returned. In the case of a single name it returns the value. In the case of a group it returns the entire group as a child object. The "f" is the footer which is an array consisting of (1) revision number, (2) status code, (3) the number of bytes pulled from RX buffer for this command (including the terminating LF), and (4) checksum (more details provided later).

**Reports** are generated automatically by the system and are therefore asynchronous. JSON reports are in the following general form:
<pre>
{"sr":{"line":0,"posx":0.000,"posy":0.000,"posz":0.000,"posa":0.000,"vel":0.000,"momo":1,"stat":3}}
</pre>

It's similar to a response except there is no header or footer element. Since it's asynchronous the status code is irrelevant, as is the number of bytes pulled form the queue (0). Since they are informational a checksum is not provided. The exception is a report that is requested, which will return a footer. E.g. the the command {"sr":""} can be used to request the status report in the above example. Look here for details of the [status reports](https://github.com/synthetos/TinyG/wiki/Status-Reports)

### JSON Syntax Option - Relaxed or Strict
JSON can be processed with either relaxed or strict syntax. `$JS` or `{js:_}` sets relaxed or strict syntax for JSON messages.
<pre>
$js=0, {js:0}   - Sets relaxed syntax
$js=1  {js:1}   - Sets strict syntax
$js    {js:n}   - Returns syntax mode
</pre>

* In strict syntax all rules follow from the [JSON spec](http://www.json.org/). All names and strings must be quoted. You can use [JSON lint](http://jsonlint.com/) to validate if your JSON expression is correctly formatted.
* In relaxed syntax names do not require quotes. String values still do.
* TinyG will accept JSON input in either relaxed or strict form regardless of the syntax setting. It will return responses in the syntax selected.
* In either syntax the shorthand `n` suffices for a null value. So this is valid: `{"fv":n}` and this `{fv:n}`
 
We recommend using relaxed mode if your parser can accept it on the responses.

###More
See [JSON Detail](https://github.com/synthetos/TinyG/wiki/JSON-Details) for more information

#Random Notes
_NOTE 1: In text mode the differences in units are obvious in the responses. In JSON there is no inherent units indication - so best to issue {"gc":"g20"} or {"gc":"g21"} at the start of every config session._

_NOTE 2: internally, everything is converted to mm mode, so if you do a bunch of settings in one units mode then change to the other the settings are still valid. Try it. Change back and forth by issuing in sequence: $x, G20, $x, G21, $x_

