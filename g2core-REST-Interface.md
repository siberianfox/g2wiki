**PRELIMINARY**<br>
This page is used for discussion of the g2core REST interface. <br>
Please also see the companion page [g2core REST Resources](g2core-REST-Resources).
_This is a design page; we expect that we will use Swagger to ultimately document the REST API._

The g2core REST API, which is the top layer of the three nested APIs:

- g2core REST API - exposed via node-g2core-server module
- g2core NodeJS API - exposed via node-g2core-api module
- g2core firmware API - exposed via serial and USB from the g2core firmware

#Background
The g2core REST API is a RESTful API that allows control of a CNC machine and associated "print jobs". The conceptual model  has these layers:

1. **Host Functions** - e.g communications setup, managing files, other non-CNC things
1. **Machine Setup** - job-agnostic machine configuration and actions (e.g. homing)
1. **Job Setup and Mgmt** - job and material setup, pre-job operations, job queuing, etc.
1. **Job Runtime** - job start and stop, pause/resume, status reporting, feedback...

### 1. Host Functions
These functions are the domain of the host operating system and don't involve the CNC machine at all. In general the g2core REST API tries not to concern itself with host functions, but some functions such as receiving, storing, and locating files are necessary to upload and manage files over the REST interface. The API may implement:

- File and filesystem primitives such as get, view, edit, write, store
- Advanced viewers such as 3d rendering

Additional host functions are not the domain of this API, may include: 
- Communications functions such as wireless setup and connectivity.
- CAD/CAM and other modeling and design

### 2. Machine Setup
Machine setup reads and writes parameters and initiates actions to configure the CNC machine independently of any given job. They may be done once and once only, periodically, or before or after a job. These may include:

- **Machine Configuration** exposes machines settings generally not changed on a per-job basis. E.g. communications settings, reporting levels, machine startup defaults. All settings exist at the same level, so it's up the the UI to determine which to show to users, which are "expert", and which should be hidden (if any). Machine configuration is implemented as synchronous REST calls to various [machine resources](g2core-REST-Resources#machine-resource).

- **Machine Operations** includes actions such as homing, axis tramming or automatic bed leveling that may be run on power up, periodically, or before a job is run. Machine operations are implemented as long-running asynchronous REST calls bundled in an [operation resource](g2core-REST-Resources#operation-resource) that provides a control context for the duration of the operation.

### 3. Job Setup and Management
Job setup and management handles commands to prepare a job for runtime. It operates on [job resources](g2core-REST-Resources#job-resource), and the job queue (job list), which orders the jobs to be run.

The model for job management is borrowed from commercial printing operations. A 'job jacket' is a container for a job. It includes one or more 'print files', a 'master' JSON doc with job metadata and a declarative control specification, and any other files that make up the job bundle such as multi-file prints, runtime logs, etc. (In implementation the jacket can be a directory containing a bunch of files).

The API implements:

- Fetch and manipulate job files
- Preview/check job file
- Queue and order jobs for execution
- Define job parameters - e.g. filament type and size

### 4. Job Runtime
Job runtime scopes those things that happen while the job is running. It is differentiated from job setup because there are commands that can only be executing during job runtime (e.g. feedholds), and other commands that should not be executed during runtime (e.g. machine configuration). The REST interface implements the following runtime state changes:

- Start/stop job (including running the Gcode)
- Pause and resume job (feedhold / cycle start)
- In-job manual operations (tool change, filament restock)
- Report job progress
- Job exception reporting, handling, and job recovery

_A Note on Formats: Most of the functions above are accessed using REST/JSON, with the Gcode commands being the exception. We call the Gcode a 'tape' because it's pre-planned motion that executes linearly, does not loop or branch (g2core does not support O codes), and can be controlled like a tape machine. It can be stopped, started, sped up and slowed down, paused and resumed. You may be able to do things in the pauses such as manual tool changes, but that doesn't change the tape itself. Tapes used to actually be paper tapes, just like phones used to have dials and dial tones._


# In-a-hurry REST interface
We have one week to get a UI up and running. We need a server-side service, and the most-obvious interface is REST. We need to support this functionality for the minimal UI:

- **One-time setup** - configuring the machine, in general.
- **Per-job setup** - this is configuring the machine for this specific job.
- **Start/pause/stop job** - what it says.
- **Upload/list/manage "job files"** - IOW, manage gcode and any metadata files we sidecar with them.
- **Status updates** - what it says.

So, let's break this down into stages of what we need more to least.

1. **Status** - if we can't see machine status, we can't do anything else.
  - This is both "state" and "status_report".
  - We should separate them so we can handle them independently. **I don't think special effort should be put into making `sr` not show `stat`, however.**
  - This is fairly simple: we can hold a "full" status report in the server as the status model, and whatever the latest status report changes we update the status model. When the status is requested, we return the status model JSON.
1. **Minimal console** - we can go a long way with a console to drive machine manually. This can be used for job setup (not ideal, but functional).
  - This implies the ability to send arbitrary gcode and JSON (possible text-mode, if that's not too much more work), and get live responses.
  - The console can display updates to the status, so to implement the console we need the ability to route JSON to the correct endpoint, and gcode to another. *Or, do we wrap console gcode in json?*
1. **Start/pause/stop**
  - This can be a state change.
    - For example, we can `GET` the state and see it is `RUN`, `READY`, `ALARM`, `HOLD`, etc. We can `PUT` the state to `HOLD` to request a feed hold and put it to `RUN` to resume.
    - Only certain transitions can be requested, and they may translate to something similar. IOW, if `HOLD` is requested during a `CYCLE`, when `RUN` is requested (since `CYCLE` shouldn't be) the returned new state would be `CYCLE`.
    - *How to clear the job? Maybe `PUT` it to `ALARM`, then `PUT` it to `READY`? Or `RUN` -> `HOLD` -> `READY`?*
1. **Upload/list/manage jobs**
  - Before this is in place, we can just manage the filesystem manually (local via Finder, or through SSH on a microhost).
  - This is fairly straightforward for a rest REST interface. We just have a `/files` endpoint, and you can `PUT` `/files/${path}` (no ending slash) to add a file, `GET` `/files/${path}/` (with a slash) to read a directory listing, or `GET` `/files/${path}` (no slash) to download a file.
1. **Per-job setup**
  - This will be things like homing, jogging, etc.
  - The urge will be strong to build this earlier, but really a console can handle these things until we are ready for this part.
1. **Machine setup**
  - This is pretty much a one-time setup per machine. This can actually be split into a diferent UI, but should share the same REST interface and server. Until this is in place, we can have a manually managed JSON file on the server that houses the JSON config for that machine that's sent during startup.
  
Ok, so with that in mind, here's a quick list of the endpoints we need, matching the order above. One note is we have to be clear in our documentation that "state" is for the machine as a whole and "status_report" is for the changeable values.

These assume an api name and version prefix in the URI. For example, `/files` below would actually be at *something like* `/g2core/v1.0/files`.

1. **Machine state** - `/status_report` and `/state`
  - `/status_report` is literally just a stored copy of the combined (complete) `sr`. *Read-Only*
  - `/state` is a (possibly interpreted) version of `stat`. As mentioned above, we could `PUT` `/state` to a value to change the machine state, and the transition requested determines the actions taken on the back end.
1. **Console** - `/config` and `/execute`
  - `PUT` `/config/${key}` with a valid JSON value will translate to a `{"key":value}` and the response will be sent back. *We need to determine format of response, but I think we take the value of the `r` key out and return that, and express the footer information separately (in the header?).*
    - This provides the console JSON capability. It could provide the gcode capability if we accept use `gc` for non-JSON code.
  - `GET` `/config/${key}` will return the JSON value of given key.
  - `PUT` `/execute/value` (content in the body) for sending small chunks of gcode. *Write-only*
  - `GET` `/execute/status` returns the status of the currently executing state, or that it's available. *Read-only*
1. **Start/stop/pause** - `/state`
  - `PUT` `/state` - request a state transition
  - `GET` `/state` - determine the current state
1. **Upload/list/manage jobs** - `/files`, `/joblist`, `/joblist/${file}`
  - *`/files` or `/file`? We need to decide on a pluralization standard and hold to it.*
  - `PUT` `/files/${path}` - (could be `POST`) upload a file at `${path}`
  - `GET` `/files/${path}` - download the file at `${path}` (**or** download file information? Maybe allow both?) 
  - `GET` `/files/${path}/` - return a JSON file listing.
  - `DELETE` `/files/${path}` - remove a file from the machine.
  - `DELETE` `/files/${path}/` - remove a directory from the machine.
  - `GET` `/joblist` - return a JSON listing of the queued jobs and their state.
  - `PUT` `/joblist` - send JSON requesting the runlist be started/cleared/etc.
    - *Job start/hold **could** be here as well. Job kill has to be.*
    - Non-current jobs can be "paused," meaning when it's next in the queue it'll be skipped and won't be executed until it's resumed.
  - `PUT` `/joblist/${path}` - request that a given file (already uploaded) be added to the runlist.
    - Response is JSON indicating if it's a valid file and can be queued.
  - `GET` `/joblist/${path}` - returns minimal running status of a given file. May be `queued`, `running`, etc.
  - `PATCH` `/joblist/${path}` - change the state of a queued job (not delete, such as move it up.)
  - `DELETE` `/joblist/${path}` - remove an item from the job queue.

#Examples

##Info
### `GET /info`

```http
REQUEST:
  GET /info HTTP/1.1

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