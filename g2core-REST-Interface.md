**PRELIMINARY - FOR DISCUSSION**<br><br>
These preliminary design pages are for discussion of the g2core REST interface: <br>

- [g2core REST Interface](g2core-REST-Interface)
- [g2core REST Resources](g2core-REST-Resources)
- [g2core REST Examples](g2core-REST-Examples)
- [g2core REST Swagger](g2core-REST-Swagger)

---
The g2core REST API is the top layer of the three nested APIs:

- g2core REST API - exposed via node-g2core-server module
- g2core NodeJS API - exposed via node-g2core-api module
- g2core firmware API - exposed via serial and USB from the g2core firmware

# Overview
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

- **Machine Settings** exposes machines settings generally not changed on a per-job basis. E.g. communications settings, reporting levels, machine startup defaults. All settings exist at the same level, so it's up the the UI to determine which to show to users, which are "expert", and which should be hidden (if any). Machine configuration is implemented as synchronous REST calls to [machine resources](g2core-REST-Resources#machine-resource).

- **Machine Operations** includes actions such as homing, axis tramming or automatic bed leveling that may be run on power up, periodically, or before a job is run. Machine operations are implemented as long-running asynchronous REST calls bundled in an [operation resource](g2core-REST-Resources#operation-resource) that provides a monitoring and control context for the duration of the operation.

### 3. Job Setup and Management
Job setup and management handles commands to prepare a job for runtime. It may operate on machine resources, perform operations, and on [job resources](g2core-REST-Resources#job-resource) and the job queue (job list), which orders the jobs to be run.

The model for job management is borrowed from commercial printing operations. A 'job jacket' is a container for a job. It includes one or more 'print files', a 'master' JSON doc with job metadata and a declarative control specification, and any other files that make up the job bundle such as multi-file prints, runtime logs, etc. (In implementation the jacket can be a directory containing a bunch of files).

The Job API implements:

- Fetch and manipulate job files
- Define job parameters - e.g. filament type and size
- Preview/check job file
- Queue and order jobs for execution

### 4. Job Runtime
Job runtime scopes those things that happen while the job is running. It is differentiated from job setup because there are commands that can only be executed during job runtime (e.g. feedholds), and other commands that should not be executed during runtime (e.g. machine configuration). The REST interface implements the following runtime state changes:

- Start/stop job (running the Gcode)
- Pause and resume job (feedhold / cycle start)
- In-job manual operations (tool change, filament restock)
- Report job progress
- Job exception reporting, handling, and job recovery

_A Note on Formats: Most of the functions above are accessed using REST/JSON, with the Gcode commands being the exception. We call the Gcode a 'tape' because it's pre-planned motion that executes linearly, does not loop or branch (g2core does not support O codes), and can be controlled like a tape machine. It can be stopped, started, sped up and slowed down, paused and resumed. You may be able to do things in the pauses such as manual tool changes, but that doesn't change the tape itself. Tapes used to actually be paper tapes, just like phones used to have dials and dial tones._
