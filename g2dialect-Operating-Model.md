This page describes the layering of the [g2dialect](g2dialect)

##Operating Model
This operating model is used to simplify and reduce the number of functions that must be handled from within Gcode. In short, we want to use Gcode only for functions that must run at Job Execution time. Setup, configuration functions, and run-time overrides (controls) are handled in JSON, or by the OS.

However, in keeping with 50+ years of CNC practice, Gcode has some practical exceptions that are noted and respected. When these exceptions occur we follow Gcode practice and try not to invent anything new.

Gcode is primarily a job description language, and much less a job control language. What Gcode does well:
- Express exact details of pre-planned motion - i.e. Gcode is static
- Initiate real-time state changes in peripherals (when these states are planned in advance)
- Sequence and synchronize complex / coordinated sets of events
	
What Gcode does not do as well:	
- Interject realtime changes into pre-planned motion (dynamic control)
- Manage devices and perform configuration (setup)
- Provide job management, beyond rudimentary start and end

JSON serves these other functions well.

## Model Layers

The key to the operating model is to remove as many dependencies from the Gcode part file as possible, and therefore make it as agnostic and reusable as it can be. The layers of the model are summarized as:

1. **OS Functions** - such as files, communications and other things that don't touch the CNC
1. **Configuration** - one-time machine parameters and some in-job configuration
1. **Job Control** - material and setup parameters, preparation steps, status reporting, runtime overrides
1. **Part File** - static file that actually runs the job

### 1. Operating System Functions
These functions are the domain of the host operating system and should not involve the CNC machine at all. They are mentioned here because they are the top-level framework in which the lower layers run. These functions include:

- File and Communications Functions
- File primitives such as get, view, edit, write
- Persistence and storage - such as disk, cloud, SD cards, etc.
- Managing web connection and other communications
- CAD/CAM and other modeling and design

The g2dialect does not implement these commands, and removes them from Mcodes and Gcodes where they are found.

Format: Native OS commands and dialogs are used.

### 2. Configuration
These are parameters and actions that set up the CNC machine regardless of the job that is to be run. These may include the following.

- **Deep Configuration** settings that define machine operation and are generally not changed per job or during a job. Examples include maximum velocity, work area sizes, axis count and configuration. These may or may not be exposed for end-user configuration.
- **End-user Configuration** settings that are set to user requirements or preference and generally not changed on a per-job basis. Examples include communications settings, reporting levels, machine startup defaults.
- **Machine Initialization** actions such as homing, axis tramming or automatic bed leveling that may be run on power up or periodically. These are also independent of any particular job.

Formats: The g2dialect performs these actions using JSON where possible, but may also use some consensus Gcode commands such as probing where necessary.

### 3. Job Control

Those job control control commands that require interaction with the CNC machine are performed using JSON. For example, Start and Stop Job may be performed using JSON.

- Job Control
  - Fetch and manipulate job files
  - Define job parameters 
  - Start/stop job
  - Pause and resume job
  - Job exception handling and recovery
  - Report on job progress / display runtime messages to users

Format: JSON is primarily used at this layer. Some functions are called using OS level commands.

### 4. Part File
This layer actually runs the part file. It interprets all Gcode commands int the file and executes the job as a sequence of time-coordinated steps. It may execute movement, heating, extrusion, cutting, laser, vacuum, or other controls for devices 

Format: The part file is Gcode. Some commands in the Gcode use [JSON active comments](JSON-active-comments) embedded in the Gcode.
