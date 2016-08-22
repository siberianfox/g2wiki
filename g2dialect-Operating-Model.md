This page describes the layering of the [g2dialect](g2dialect)

##Operating Model
This operating model is used to simplify and reduce the number of functions that must be handled from within Gcode. In short, we want to use Gcode only for functions that must run at Job time. Setup, configuration functions, and run-time overrides (controls) are handled in JSON or by other non-Gcode channels.

However, in keeping with 50+ years of CNC practice, Gcode has some practical exceptions that are noted and respected. When these exceptions occur we follow Gcode practice and try not to invent anything new.

Gcode is primarily a job description language, and much less a job control language. What Gcode does well:
- Express exact details of pre-planned motion - i.e. Gcode is static
- Initiate real-time state changes in peripherals (when these states are planned in advance)
- Sequence and synchronize complex / coordinated sets of events
	
What Gcode does not do as well:	
- Interject realtime changes into pre-planned motion (dynamic control)
- Manage devices and perform configuration (setup)
- Provide job management, beyond rudimentary START, STOP and END functions

JSON can be made to serve these other functions well.

## Model Layers

The key to the operating model is to remove as many dependencies from the Gcode part file as possible, and therefore make it as agnostic and reusable as it can be. The layers of the model are summarized as:

1. **Host Functions** - such as files, communications and other things that don't touch the CNC
1. **Machine Setup** - one-time machine configuration - independent of a given job
1. **Job Setup** - job and material setup parameters, preparation steps, and any pre-job operations
1. **Job Runtime** - Job start and stop, pause/resume, overrides; status reporting and real-time feedback

### 1. Host Functions
These functions are the domain of the host operating system and should not involve the CNC machine at all. They are mentioned here because they are the top-level framework in which the lower layers run. These functions include:

- File and Communications Functions
- File primitives such as get, view, edit, write
- Persistence and storage - such as disk, cloud, SD cards, etc.
- Managing web connection and other communications
- CAD/CAM and other modeling and design

The g2dialect does not implement these commands, and removes them from Mcodes and Gcodes where they are found.

Format: Native OS commands and dialogs are used.

### 2. Machine Setup
These are parameters and actions that set up the CNC machine regardless of the job that is to be run. They may be done once and once only, periodically, or before a job that is to be run. These may include the following.

- **End-User Configuration** includes settings that are accessible to users and generally are not changed on a per-job basis. Examples include communications settings, reporting levels, machine startup defaults.
- **Deep Configuration** includes settings that define machine operation, are generally not of interest to users, and are typically not changed per job or during a job. Examples include maximum velocity, work area sizes, axis count and configuration. 
- **Machine Initialization** includes settings and actions such as homing, axis tramming or automatic bed leveling that may be run on power up or periodically. These are also independent of any particular job.

Formats: The g2dialect performs these actions using JSON where possible, but may also use some consensus / historical Gcode commands such as coordinate system offsets and probing where necessary.

### 3. Job Setup
Includes commands that are used to prepare the machine for a specific job. These may include

- Fetch and manipulate job files
- Preview/check job file
- Define job parameters - e.g. filament type and size

### 4. Job Control
Those things that happen while the job is running - including staring and stopping the job.
- Start/stop job (run the Gcode)
- Pause and resume job
- Job exception handling and job recovery
- Report on job progress
- Display runtime messages to users

Format: JSON is primarily used at this layer. Some functions are called using OS level commands.
