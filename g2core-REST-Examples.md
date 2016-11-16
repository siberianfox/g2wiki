**PRELIMINARY - FOR DISCUSSION**<br><br>
These preliminary design pages are for discussion of the g2core REST interface: <br>

- [g2core REST Interface](g2core-REST-Interface)
- [g2core REST Resources](g2core-REST-Resources)
- [g2core REST Examples](g2core-REST-Examples)
- [g2core REST Swagger](g2core-REST-Swagger)

---

#g2core REST API Examples

##Version
Provides an endpoint for system and API version information that is independent of the machine or any other resource
### `GET /version`

```http
REQUEST:
  GET /version HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"fv":0.99,"fb":100.12,"fbs":"100.12-14-g375c-dirty","fbc":"settings_makeblock.h","hp":3,"hv":0,"id":"0084-7bd6-29c6-8ce","msg":"SYSTEM READY"}

```

## Machine
The machine REST endpoint is used to query machine state and to query and set machine parameters in the [machine resource](g2core-REST-Resources#machine-resource). It is intended for JSON commands that execute synchronously, as most queries and configuration settings do. The API can handle single values, JSON objects, or composite JSON documents consisting of multiple values and/or objects. Examples are provided for each.

To send Gcode or to invoke long-running JSON commands that require asynchronous handling see [operation](#operations).

### `GET /machine`
Return machine objects and/or attributes by a JSON specification in the request body. The following calling styles are supported:

```
(body omitted)              no body provided. Return entire machine
{ }                         key is empty object. Return entire machine
{"stat":null}               key is freestanding; not part of an object (system state)
{"x":null}                  key is an object (X axis)
{"x":{"vm":null}}           key is part of an object (X axis max velocity)
{"xvm":null}                key is flattened version of above (may be deprecated later)
{"x.vm":null}               key is flattened version of above (planned for future)
{"x":null,"y":null}         key described multiple objects (X and Y axes)
{"x":null,"y":{"jm":null}}  key is mixed objects and attributes
{"posx":null,"posy":null,"posz":null,"vel":null,"stat":null} key has multiple attributes
```

```http
REQUEST:
  GET /info HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
{"value":
{
"sys":{"fb":100.12,"fbs":"100.12-14-g375c-dirty", "fbc":"settings_makeblock.h","fv":0.99,"hp":3,"hv":0, "id":"0084-7bd6-29c6-4ad", "jt":0.75,"ct":0.1,"sl":0,"lim":0,"saf":1,"m48e":1,"mfoe":0,"mfo":1,"mtoe":0,"mto":1,"mt":2,"spep":1,"spdp":0,"spph":1,"spdw":1,"ssoe":0,"sso":1,"cofp":1,"comp":1,"coph":0,"tv":1,"ej":1,"jv":2,"qv":0,"sv":1,"si":250,"gpl":0,"gun":1,"gco":1,"gpa":2,"gdi":0},
"x":{"am":1,"vm":40000,"fr":40000,"tn":0,"tm":420,"jm":5000,"jh":20000,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":4,"zb":2},
"y":{"am":1,"vm":40000,"fr":40000,"tn":0,"tm":420,"jm":5000,"jh":20000,"hi":3,"hd":0,"sv":3000,"lv":100,"lb":4,"zb":2},
"z":{"am":1,"vm":1200,"fr":1200,"tn":-95,"tm":0,"jm":500,"jh":1000,"hi":6,"hd":1,"sv":800,"lv":25,"lb":4,"zb":2},
"a":{"am":0,"vm":393701,"fr":393701,"tn":-1,"tm":-1,"jm":49213,"jh":49213,"ra":5.821,"hi":0,"hd":0,"sv":196850,"lv":39370.08,"lb":5,"zb":2},
"b":{"am":0,"vm":393701,"fr":393701,"tn":-1,"tm":-1,"jm":49213,"jh":49213,"ra":5.821,"hi":0,"hd":0,"sv":196850,"lv":39370.08,"lb":5,"zb":2},
"c":{"am":0,"vm":393701,"fr":393701,"tn":-1,"tm":-1,"jm":49213,"jh":49213,"ra":5.821,"hi":0,"hd":0,"sv":196850,"lv":39370.08,"lb":5,"zb":2},
"1":{"ma":0,"sa":1.8,"tr":36.576,"mi":8,"su":43.74453,"po":0,"ep":1,"pm":2,"pl":0.4},
"2":{"ma":1,"sa":1.8,"tr":36.576,"mi":8,"su":43.74453,"po":0,"ep":1,"pm":2,"pl":0.4},
"3":{"ma":2,"sa":1.8,"tr":1.25,"mi":8,"su":1280,"po":0,"ep":1,"pm":2,"pl":0.4},
"4":{"ma":3,"sa":1.8,"tr":360,"mi":8,"su":4.44444,"po":0,"ep":1,"pm":0,"pl":0},
"do1":{"mo":1},
"do2":{"mo":1},
"do3":{"mo":1},
"do4":{"mo":1},
"do5":{"mo":1},
"do6":{"mo":1},
"do7":{"mo":1},
"do8":{"mo":1},
"do9":{"mo":1},
"di1":{"mo":0,"ac":0,"fn":0},
"di2":{"mo":0,"ac":0,"fn":0},
"di3":{"mo":0,"ac":0,"fn":0},
"di4":{"mo":0,"ac":0,"fn":0},
"di5":{"mo":0,"ac":0,"fn":0},
"di6":{"mo":0,"ac":0,"fn":0},
"di7":{"mo":0,"ac":0,"fn":0},
"di8":{"mo":0,"ac":0,"fn":0},
"di9":{"mo":0,"ac":0,"fn":0},
"he1":{"e":false,"p":9,"i":0.12,"d":400,"st":0,"t":-1,"op":0,"tr":-1,"at":false,"an":0,"fp":1,"fm":0.4,"fl":40,"fh":40},
"he2":{"e":false,"p":9,"i":0.12,"d":400,"st":0,"t":-1,"op":0,"tr":-1,"at":false,"an":0,"fp":1,"fm":0.4,"fl":40,"fh":40},
"he3":{"e":false,"p":9,"i":0.12,"d":400,"st":0,"t":-1,"op":0,"tr":-1,"at":false,"an":0,"fp":1,"fm":0.4,"fl":40,"fh":40},
"g54":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g55":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g56":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g57":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g58":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g59":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g92":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g28":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
"g30":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0},
}
}
```

Return top level objects in a machine
```http
REQUEST:
  GET /info?shallow=true HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":{"sys":true,"x":true,"y":true,"z":true,"a":true,"b":true,"c":true,"1":true,"2":true,"3":true,"4":true,"do1":true,"do2":true,"do3":true,"do4":true,"do5":true,"do6":true,"do7":true,"do8":true,"do9":true,"di1":true,"di2":true,"di3":true,"di4":true,"di5":true,"di6":true,"di7":true,"di8":true,"di9":true,"he1":true,"he2":true,"he3":true,"g54":true,"g55":true,"g56":true,"g57":true,"g58":true,"g59":true,"g92":true,"g28":true,"g30":true}}
```
Response prettified:
```
{
	"value": {
		"sys": true,
		"x": true,
		"y": true,
		"z": true,
		"a": true,
		"b": true,
		"c": true,
		"1": true,
		"2": true,
		"3": true,
		"4": true,
		"do1": true,
		"do2": true,
		"do3": true,
		"do4": true,
		"do5": true,
		"do6": true,
		"do7": true,
		"do8": true,
		"do9": true,
		"di1": true,
		"di2": true,
		"di3": true,
		"di4": true,
		"di5": true,
		"di6": true,
		"di7": true,
		"di8": true,
		"di9": true,
		"he1": true,
		"he2": true,
		"he3": true,
		"g54": true,
		"g55": true,
		"g56": true,
		"g57": true,
		"g58": true,
		"g59": true,
		"g92": true,
		"g28": true,
		"g30": true
	}
}
```

### `GET /machine/{key}`
A `{key}` can be any of the following. The examples illustrate these cases.

- `/machine/stat` - key is freestanding; not part of an object (system state)
- `/machine/x/vm` - key is part of an object (X axis max velocity)
- `/machine/xvm` - flattened version of the above (may be deprecated in the future)
- `/machine/x.vm` - flattened version of the above planned for future
- `/machine/x` - key is entire object

Get the value of a single `{key}` of `ct` (chordal tolerance) as an example, which is a numeric type.
```http
REQUEST:
  GET /machine/ct HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":0.01}
```

Get the value of a single `{key}` `x/jm` (max jerk in the X axis), which is a numeric type.
```http
REQUEST:
  GET /machine/x/jm HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":1200}
```

Same as above using flattened notation `xjm`
```http
REQUEST:
  GET /machine/xjm HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":1200}
```

Get the value of a single object `{key}` using the `pos` object as an example.
```http
REQUEST:
  GET /machine/pos HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":{"pos":{"x":0,"y":0,"z":0,"a":0,"b":0}}}
```

### `PUT /machine/{key}`
Using a `{key}` of `xjm` as an example, which is a numeric type.
```http
REQUEST:
  GET /machine/xjm HTTP/1.1
  {"value":1200}

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":1200}
```

Using a `{key}` of `g54` as an example, which is an object type.
```http
REQUEST:
  GET /machine/pos HTTP/1.1
  {"value":{"g54":{"x":0,"y":0,"z":0,"a":0,"b":0}}}

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"value":{"g54":{"x":0,"y":0,"z":0,"a":0,"b":0}}}
```

## Status Reports
### `GET /status_report`
See [status report resource](g2core-REST-Resources#status_report-resource) for operation.

_Question: Is it useful for status_report to report back the operation ID and/or job ID it is reporting on?_

Use a status report with no filter to return all keys configured in the status report
```http
REQUEST:
  GET /status_report HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"line":101,"posx":10,"posy":20,"posz":-1,"posa":0,"posb":0,"posc":0,"feed":1000,"vel":1000,"momo":0,"stat":5}
```

Use a status report filter with null values to select which keys to return
```http
REQUEST:
  GET /status_report HTTP/1.1
  {"posx":null,"posy":null,"posz":null,"vel":null,"stat":null}

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"posx":10,"posy":20,"posz":-1,"vel":1000,"stat":5}}
```

Use a status report filter with non-null values to return only keys that have different values (have changed) 
```http
REQUEST:
  GET /status_report HTTP/1.1
  {"posx":10,"posy":20,"posz":-1,"vel":1000,"stat":5}

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"posx":10.2}
```

## Operations
Enums:

- Operation types (optype)
  - run - run arbitrary Gcode, JSON or a mix
  - sendfile - send a file
- Operation states (opstate)
  - pend
  - run
  - done
  - error

### `POST /operation`
Start a new RUN operation to execute multiple Gcode lines<br>

The following are all valid ways to invoke a RUN operation:

- ``

Option 1 - pipe delimited string
```http
REQUEST:
  POST /operation HTTP/1.1
  {"optype":"run","value":"G28.2 z0 | G28.2 x0 y0"}
  {"optype":"run", "value":["G28.2 z0","G28.2 x0 y0"]}
```

Option 2 - array type
```http
REQUEST:
  POST /operation HTTP/1.1
  {"optype":"run"},
  {"value":["G28.2 z0","G28.2 x0 y0"]}
```

Option 3 - multiple Value keys. Can you even do this? Do they remain ordered?)
```http
REQUEST:
  POST /operation HTTP/1.1
  {"optype":"run"},
  {"value":"G28.2 z0"
  {"value":"G28.2 x0 y0"} 
```

```http
RESPONSE:
  HTTP/1.x 201 RESOURCE HAS BEEN CREATED
  Content-Type: application/json; charset=UTF-8
  42
```

### `GET /operation/{id}`
Get a current operation

```http
REQUEST:
  GET /operation/42 HTTP/1.1

RESPONSE:
  HTTP/1.x 200 OK
  Content-Type: application/json; charset=UTF-8
  {"opstate":"run"},
  {"value":"??? What does it really return?"}
```

### `PUT /operation/{id}`


## Jobs
### `POST /job`

### `GET /job/{id}`

### `PUT /job/{id}`

## Error Responses
All error responses look like this:

```http
HTTP/1.x 404 NOT FOUND
Content-Type: application/json; charset=UTF-8
{"status":{"code":100,"description":"Unrecognized command or config name","additional_description":"xyzzy is not supported at the current time"}}
```

### Commonly Used HTTP Response Codes
Defines the use of HTTP response codes and the mapping from g2core stat codes to HTTP codes

#### Success Codes

- 200 OK
  - STAT_OK 0
- 201 Created
- 202 Accepted

#### Exception Codes

- 400 Bad Request
  - Syntax errors 101-103, 111-116, 200-203, 207-209
  - Gcode specification errors 130-181
  - Homing errors 240-247
  - Probing errors 250-254
- 404 Not Found
  - STAT_UNRECOGNIZED_NAME 100
  - STAT_NO_SUCH_DEVICE 11
- 405 Method Not Allowed
  - STAT_PARAMETER_IS_READ_ONLY 104
  - STAT_PARAMETER_CANNOT_BE_READ 105
  - STAT_COMMAND_NOT_ACCEPTED 106
  - Soft limit errors 220-233
- 408 Request Timeout
  - from host
- 410 Gone
  - STAT_SHUTDOWN 5
  - STAT_PANIC 6
  - STAT_ALARM 18
  - STAT_COMMAND_REJECTED_BY_ALARM 204
  - STAT_COMMAND_REJECTED_BY_SHUTDOWN 205
  - STAT_COMMAND_REJECTED_BY_PANIC 206
- 413 Request Entity Too Large
  - STAT_FILE_SIZE_EXCEEDED 10
  - STAT_INPUT_EXCEEDS_MAX_LENGTH 107
- 416 Requested Range Not Satisfiable
  - STAT_INPUT_LESS_THAN_MIN_VALUE 108
  - STAT_INPUT_EXCEEDS_MAX_VALUE 109
  - STAT_INPUT_VALUE_RANGE_ERROR 110

- 500 Internal Server Error
  - Internal errors 7-9, 12-17, 19-36, 88-99
- 501 Not Implemented
  - from host
- 504 Gateway Timeout
  0 from host
