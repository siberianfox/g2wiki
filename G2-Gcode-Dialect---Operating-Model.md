##Operating Model
The following operating model is used to simplify and reduce the number of functions that must be handled from within Gcode. In short, use Gcode only for functions that must run at "Job" time. Setup, configuration functions, and run-time overrides (controls) are handled in JSON. 

However, in keeping with 50+ years of CNC practice, Gcode has some exceptions to these (perhaps too neat) categories that are noted. When that occurs we follow Gcode practice and try not to invent anything new.

##Model layers

- Job control and high-level user interface functions

		Job control functions - e.g. define job parameters, Start/stop job, report on job progress, display runtime messages to users
		File handling functions - e.g. get a file, run a file, SD card functions, etc.
		Web communications - e.g. HTML5, config pages, etc.
		
JSON	2)	Configuration and job setup
		Things that may happen before a job, but should never happen during a job, like:
		Configuring - machine parameters, extruders, motors, system variables, etc.
		Machine functions that run in advance of a job, like homing or bed leveling
		
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