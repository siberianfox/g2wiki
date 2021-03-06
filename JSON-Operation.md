This page is the start page describing JSON operation in G2.

Related Pages:
- [JSON Details](JSON-Details)
- [JSON Active Comments](JSON-Active-Comments)
- [JSON Cheat Sheet](JSON-Cheat-Sheet)

## General

G2 can accept commands in [JSON](http://json.org/) (JavaScript Object Notation) mode or text mode. Using JSON simplifies writing host GUIs in languages such as Python, Ruby, JavaScript, Java, Processing and other languages that support dictionaries / hashmaps.

Most commands that are available in JSON are also available in text mode, but there are some differences. ASCII communications overhead is somewhat higher in JSON than text mode but is still quite efficient and manageable.

#### Syntax Notes
- The examples above are in strict JSON mode, as specified by json.org. TinyG will also accept input in relaxed JSON mode which omits the quotes on all fields except if the value field is a string. All responses are always in strict mode.
- The following values can be used in full or by their abbreviations:
  - `null` or `n`
  - `true` or `t`
  - `false` or `f`

### Is this RESTful?
The JSON interface is modeled as a RESTful interface, albeit running over USB or serial and not HTTP. Using JSON enables exposing system internals as [RESTful resources](http://en.wikipedia.org/wiki/Representational_state_transfer), which makes the embedded system much easier to manipulate using modern programming techniques. Data is treated as resources and state is transferred in and out. Unlike RESTful HTTP, methods are implied by convention as there is no request header in which to declare them.

## Basic Concepts
In JSON mode G2 expects well structured JSON (if in doubt use the [JSON validator](http://jsonlint.com)). JSON requests are generally just curly braces and formatting around what would otherwise be a command line request. This may be a single value such as {"xvm":null}, or a complete group such as {"x":null}. Only one parent JSON object is processed per line. Note that groups with multiple *elements* are accepted. For example, this is OK:
{"x":{"vm":16000,"fr":10000}}

### How things are encoded in JSON

- **name** is a JSON key (aka **token**) describing a single datum or group of data values
  - Example: `xfr` referring to the X axis maximum feed rate
  - Example: `x` referring to all values associated with the X axis (the X axis group)
  - Names are not case sensitive
- **value** is a number, a quoted string, true/false, or null (as per JSON spec)
  - True and false values can be `true` and `false` or `t` and `f` for short
  - NULL values can be `null` (case insensitive), or simply `n` for short
  - Null values are GETs, all others will set the value, or in some cases invoke an action
  - A null value in a response indicates that the value in invalid (e.g trying to read a switch that is not configured)
- **NVpair** is a name:value pair or NV pair
- **group** is a collection of one or more NV pairs
  - Groups are used to specify all parameters for a motor, an axis, a PWM channel, or other logical grouping
  - A group is similar in concept to a RESTful resource or composite.

### What is encoded in JSON

Term | Description
---------------|--------------
**config** | A config is a static configuration setting for some aspect of the machine. These parameters are not changed by Gcode execution (but see the G10 exception). {`xfr:1000}` is an example of a config. So is `{1po:1}`. So is the X group: `{x:n}`.
**block** | Gcode blocks are lines of gcode consisting of one or more gcode words, optional comments and possibly gcode messages
**word** | Gcode words make up gcode commands. `G1` is an example of a gcode word. So is `x23.43`
**comment** | A Gcode comment is denoted by parentheses - `(this is a gcode comment)`
**JSON active comment** | A [JSON active comment](JSON-Active-Comments) is a way to insert JSON in a Gcode stream
**message** | A **Gcode message** is a special form of active comment that is echoed to the machine operator. It's the part of the comment that follows a `(msg` preamble. For example: `(msgThis part is echoed to the user)`

## JSON Overview & TinyG Subset

The concise JSON language definition is [here](http://json.org). [Jsonlint](http://jsonlint.com) is a handy JSON validator that you can use to check your requests or responses.

TinyG implements a subset of JSON with the following limitations:

* Supports 7 bit ASCII characters only
* Supports decimal numbers only (no hexadecimal numbers or other non-decimals)
* Arrays are returned but are not (yet) accepted as input
* Names (keys) are case-insensitive and cannot be more than 5 characters
* Groups cannot contain more than 24 elements (name/value pairs)
* JSON input objects cannot exceed 254 characters in total length. Outputs can be up to 512 chars.
* Limited object nesting is supported (you won't typically see more than 3 levels)
* All JSON input and output is on a single text line. There is only one `<LF>`, it's at the end of the line (broken lines are not supported)

## JSON Request and Response Formats
JSON requests are used to perform the following actions {with examples}

* Return the value of a single setting or state variable `{"1mi":n}`
* Return the values of a group of settings or state variables (aka a Resource) `{"1":n}`
* Set a single setting or state variable (note that many state variables are read-only) `{"1mi":8}`
* Set a multiple settings or state variables in a group `{"1":{"po":1,"mi":8}}`
* Submit a block (line) of Gcode to perform any supported Gcode command `{"gc":"n20g1f350 x23.4 y43.2"}`. Don't send all your Gcode this way as it can cause problems, see `note 3` for more information.
* Special functions and actions;
 * Request a status report `{"sr":n}`
 * Set status report contents `{"sr":{"line":true,"posx":true,posy":true,   ...}}`
 * Run self tests `{"test":1}`
 * Reset parameters to defaults `{"defa":true}`

JSON responses to commands are in the following general form.
<pre>
{"xjm":n} returns:
{"r":{"xjm":5000000000.000},"f":[3,0,6]}

{"2":n} returns:
{"r":{"2":{"ma":1,"sa":1.800,"tr":36.540,"mi":8,"po":1,"pm":1}},"f":[3,0,6]}
</pre>

The `r` is the response envelope. The body of the response is the result returned. In the case of a single name it returns the value. In the case of a group it returns the entire group as a child object. 

The `f` is the footer which is an array consisting of (1) revision number, (2) status code, (3) the number of lines available in the line buffers.

### Status Reports
Status reports are generated automatically by the system and are therefore asynchronous. JSON reports are in the following general form:
<pre>
{"sr":{"line":0,"posx":0.000,"posy":0.000,"posz":0.000,"posa":0.000,"vel":0.000,"momo":1,"stat":3}}
</pre>

It's similar to a response except there is no header or footer element. Since it's asynchronous the status code is irrelevant, as is the number of lines available in the buffer (0). The exception is a status report that is requested, which will return a footer. E.g. the the command `{sr:n}` can be used to request the status report in the above example. Look here for details of the [status reports](Status-Reports)

### Exception Reports
Exception reports are generated by the system when something wrong is detected.
The information provided is:
- `fb` firmware build number
- `st` status code of the exception
- `msg` a displaytable, human readbale message describing the exception

Exception reports are in the following general form:
<pre>
{"er":{"fb":100.10,"st":29,"msg":"Generic exception report - bogus exception report"}}
</pre>


### JSON Syntax handling - Relaxed or Strict
JSON can be processed with either relaxed or strict syntax. The system will always accept either, and will always return responses and reports in strict mode.

* In strict syntax all rules follow from the [JSON spec](http://www.json.org/). All names and strings must be quoted. You can use [JSON lint](http://jsonlint.com/) to validate if your JSON expression is correctly formatted.
* In relaxed syntax names do not require quotes. String values still do.
* TinyG will accept JSON input in either relaxed or strict form regardless of the syntax setting. It will always return responses in strict mode.
* In either syntax the following shorthands apply:
  - `n` suffices for a null value. So this is valid: `{"fv":null}` and this `{fv:n}`
  - `t` suffices for a true value
  - `f` suffices for a false value

### More
See [JSON Detail](JSON-Details) for more information

# Random Notes
_NOTE 1: In text mode the differences in units are obvious in the responses. In JSON there is no inherent units indication - so best to issue {"gc":"g20"} or {"gc":"g21"} at the start of every config session._

_NOTE 2: internally, everything is converted to mm mode, so if you do a bunch of settings in one units mode then change to the other the settings are still valid. Try it. Change back and forth by issuing in sequence: $x, G20, $x, G21, $x_

_NOTE 3: Sending Gcode wrapped in `{"gc": ""}` can cause an overflow in the motion planer buffer when using linemode protocol. Instead send you Gcode as plain text (while still observing linemode protocol), this is also more efficient in terms of serial bandwidth. The issue is caused by all json commands being seen as control instructions which are immediately acknowledged. See [Issue 287](https://github.com/synthetos/g2/issues/287) for an in depth discussion of this._