##Operating Model
The following operating model is used to simplify and reduce the number of functions that must be handled from within Gcode. In short, use Gcode only for functions that must run at "Job" time. Setup, configuration functions, and run-time overrides (controls) are handled in JSON. 

However, in keeping with 50+ years of CNC practice, Gcode has some practical exceptions that are noted and respected. When these exceptions occur we follow Gcode practice and try not to invent anything new.

## Model Layers

Summarized:

1. **OS Functions** - such as files, communications and other things that don't touch the CNC
1. **Machine Configuration** - one-time machine parameters and some in-job configuration
1. **Job Control** - material and setup parameters, preparation steps, status reporting, runtime overrides
1. **Canned Job** - static file that actually runs the job ("the tape")

### 1. Operating System Functions
These functions are the domain of the host operating system and should not involve the CNC machine at all. They are mentioned here because they are the top-level framework in which the lower layers run. g2dialect also removes these from Mcodes, as many have crept in over time. These functions include:

- File and Communications Functions
  - CAD/CAM aand other modeling and design
  - File primitives such as get, view, edit, write
  - Persistence and storage - such as disk, cloud, SD cards, etc.
  - Managing web connection and other communications


### 2. Machine Configuration
These are parameters and actions that set up the CNC machine regardless of the job that is to be run. This may include 
### 3. Job Control

Those job control control commands that require interaction with the CNC machine are performed using JSON. For example, Start and Stop Job may be performed using JSON.

- Job Control
  - Fetch and manipulate job files
  - Define job parameters 
  - Start/stop job
  - Report on job progress / display runtime messages to users

### 4. Canned Job

JSON	2)	Configuration and job setup
		Things that may happen before a job, but should never happen during a job, like:
		Configuring - machine parameters, extruders, motors, system variables, etc.
		Machine functions that run in advance of a job, like homing or bed leveling

Machine Configuration and Job Setup

Gcode	3)	The job, aka "the tape"
		Things that are pre-defined to execute in sequence as part of the job:
		The job itself, consition of movement commands, cutting, extrusion 
		Other items that rae synchronized with or gated by motion: temperature changes and waits, coolant, vacuums, cameras, etc.
		Some in-cycle coordinated commands plan to a stop, some do not


JSON	4)	Operator overrides that occur during the job
		Things that occur asynchronously with the job such as speed overrides, pauses & restarts (feedholds & cycle starts)... 
		
		
Gcode 		
	A gcode file describes a static series of steps that are performed in sequence at runtime (a job)	
	Gcode is primarily a job description language, and much less a job control language	
	What Gcode does well (at the risk of getting too theoretical):	
		Express exact details of pre-planned motion - i.e. Gcode is static
		Initiate real-time state changes in peripherals (when these states are planned in advance)
		Sequence and synchronize complex / coordinated sets of events
	What Gcode does not do as well:	
		Interject realtime changes into pre-planned motion (dynamic control)
		Manage devices and perform configuration (setup)
		Provide job management, beyond rudimentary start and end
		
JSON		
	A JSON object holds a collection of related parameters and may also have action commands (methods)	
		Considerations for using JSON: (+++to be completed)
		+++cover namespaces and namespace isolation
		+++cover using namespaces for extensibility & also backwards compatible design
		+++cover methods and how to express and invoke them
		+++cover distributed (chunked) parsing and object decomposition
		+++cover object routing and routability and how this supports distributed systems