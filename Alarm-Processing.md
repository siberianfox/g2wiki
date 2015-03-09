##Machine State and Alarm Processing
###Machine States
A minor addition to the combined machine state is the addition of the Interlock state. Some other states have been slightly redefined. The ALARM state refers to both hard and soft alarms. Limits no longer cause SHUTDOWN, which is now invoked by an estop event. Here are the revised machine states (from canonical_machine.h)

<pre>
enum cmCombinedState {
	COMBINED_INITIALIZING,	// [0] machine is initializing
	COMBINED_READY,		// [1] machine is ready for use
	COMBINED_ALARM,		// [2] machine in alarm state (soft or hard)
	COMBINED_PROGRAM_STOP,	// [3] program stop/no more blocks
	COMBINED_PROGRAM_END,	// [4] program end
	COMBINED_RUN,		// [5] motion is running
	COMBINED_HOLD,		// [6] motion is holding
	COMBINED_PROBE,		// [7] probe cycle active
	COMBINED_CYCLE,		// [8] DEPRECATED: machine is cycling, now just COMBINED_RUN
	COMBINED_HOMING,	// [9] homing cycle 
	COMBINED_JOG,		// [10] jogging cycle active
	COMBINED_SHUTDOWN,	// [11] machine in shutdown (due to emergency stop)
	COMBINED_INTERLOCK	// [12] machine in interlock hold
};
</pre>

####Alarm Enables
There are a few settings that enable and disable various alarm cases:
•	{sl:1} Soft limit enable: set to 0 or 1
•	{lim:1} Hard limit enable: set to 0 or 1
•	{ilck:1} Interlock enable: set to 0 or 1

Setting hard limit enable to 0 is the same as a limit override. This can be used to drive the system off a limit switch. Note that if the system is in an alarm state the alarm must be cleared prior to changing this value. {clear:n}

###Alarm Use Cases
The following use cases are supported:
•	Soft Limit Case: A Gcode block is received that would exceed the maximum or minimum travel in one or more axes. This could be true because the cutting path exceeds the available machine extents, or because the part was located (zeroed) on the table incorrectly (e.g. “not centered”). In either case assume the job is not recoverable. Soft limits are only applied if the machine is homed. The desired behavior is triggered as soon as the Gcode block is interpreted, and is:
o	Transition to ALARM state (from RUN)
o	Stop movement while preserving position (feedhold to a STOP)
•	If already in feedhold do not execute the STOP
o	Stop spindle or other actuator (extruder, laser, torch…)
o	No change to coolant output
o	Motor power timeouts activate based on motor power settings – e.g. only-when-moving or in-cycle
o	Flush the planner queue 
o	Drain the serial queues: Read and reject motion and SET commands (See CLEAR command).
o	Generate exception report for SOFT_LIMIT w/line number of the limit (if Gcode has line numbers)
o	Transition to PROGRAM_END state on receipt of CLEAR command

•	Limit Switch Hit / Unrecoverable Case: A limit switch has been hit. The input has been configured so that hitting that limit switch on that axis is unrecoverable, as position information will have been lost. The desired behavior is:
o	Transition to HARD_ALARM state (from RUN)
o	Stop movement without preserving position (STOP_STEPPING)
•	Do not perform this action already in feedhold
o	Stop spindle or other actuator (extruder, laser, torch…)
o	No change to coolant output
o	Motor power timeouts activate based on motor power settings – i.e. no longer moving or in cycle
o	Flush the planner queue
o	Drain the serial queues: Read and reject motion and SET commands (See CLEAR command).
o	Mark the limit axis as UNHOMED and mark the machine as UNHOMED
o	Generate exception report for LIMIT HIT, and indicate which input was tripped
o	Transition to PROGRAM_END state on receipt of CLEAR command 
o	Accept LIMIT_OVERRIDE to disable limits before moving machine off limit switch
o	Host should remove LIMIT_OVERRIDE before the next cycle

•	Limit Switch Hit / Recoverable Case: A limit switch has been hit. The input has been configured so that hitting that limit switch on that axis is potentially recoverable, as position information has been preserved. The desired behavior is:
o	Transition to SOFT_ALARM state (from RUN)
o	Stop movement with rapid deceleration while preserving position (HALT)
•	If already in feedhold do not execute the HALT
o	Stop spindle or other actuator (extruder, laser, torch…)
o	No change to coolant output
o	Motor power timeouts activate based on motor power settings – i.e. no longer moving or in cycle
o	Generate exception report for LIMIT HIT, and indicate which input was tripped
o	Transition to PROGRAM_END state on receipt of CLEAR command or ~ char (cycle start)
o	Accept LIMIT_OVERRIDE to disable limits before moving machine off limit switch
o	Host should remove LIMIT_OVERRIDE before the next cycle
o	Host is responsible for restarting spindle if job is recovered

•	Interlock Hold Case: An interlock condition has been detected on an input. Interlocks should allow jobs to resume once the interlock condition has been removed. The desired behavior is:
o	Transition to INTERLOCK state (from RUN)
o	Stop movement with rapid deceleration while preserving position (HALT into feedhold)
•	If already in feedhold do not execute the HALT
o	Stop spindle or other actuator (extruder, laser, torch…)
o	No change to coolant output
o	Motor power timeouts activate based on motor power settings – i.e. no longer moving or in cycle
•	Q: Or do we want them to stay energized indefinitely?
o	Generate exception report for entering INTERLOCK condition w/the interlock input number
o	Wait for interlock condition to be cleared by the input.
o	Generate exception report for exiting INTERLOCK condition w/the interlock input number
o	Restore spindle to prior state, with delay dwell of N seconds (settable)
o	Resume motion by resuming from feedhold (CYCLE_START)
•	…unless the board was already in feedhold at the time of INTERLOCK

•	Emergency Shutdown – Case: Emergency shutdown is the controller’s reaction to an EStop being activated. Note that Estop is NOT a controller function, it is an external condition that must remove power and/or brake all moving parts, including axes, spindles, etc. This occurs outside of the controller’s domain. The Emergency Shutdown input is used to tell the controller that this condition has occurred, and for it to take appropriate steps on its own. Emergency Shutdown is not recoverable. The desired behavior is:
o	Transition to SHUTDOWN state (from RUN)
o	Stop movement with deceleration without preserving position (STOP_STEPPING)
o	Stop spindle or other actuator (extruder, laser, torch…)
o	Stop coolant output
o	Disable motor power (no timeout)
o	Flush the planner queue
o	Mark all axes as UNHOMED and mark the machine as UNHOMED
o	Drain the queues: Read and reject motion and SET commands (See CLEAR command).
o	Exception report is generated reporting SHUTDOWN condition
o	Transition to PROGRAM_END state on receipt of CLEAR
o	Note: The external Estop logic or the host may decide to reset the controller

Open Questions:
•	Do we need to have some kind of a mask for what to do about digital outputs in these cases – both trigger and restore? Ultimately flexible, but lots of complexity in configuration. Or rely on the host to perform this somehow. The response from CLEAR should be the trigger for the host to take more actions if they want to.
•	Recoverable limit still has problems. How to you get the CLEAR ahead of the other commands? We can manage this on the board with a ~, and can probably do this on G2, but v8 and configs with upstream buffering may have problems with this.

#### CLEAR Command
When entering an alarm state there may be Gcode and configuration commands buffered in multiple places including on-board serial receive queues (RX), board and host internal USB queues, sender queue (e.g. serial-port-json-server), terminal emulator or host transmit buffer queues, or other upstream buffers. 
In an unrecoverable alarm it is desirable to cease all motion, so this requires rejecting all new commands that may be received after the alarm condition is triggered. Commands received in a HARD_ALARM or SHUTDOWN state will be “eaten” and not processed, and will return with status code 203, STAT_MACHINE_ALARMED. 
In this state it is possible to query g2, (i.e. GET commands), but not possible to change the state of the machine. 
(Q: do we want to let SR requests get through? Probably. Test this.)
Once a CLEAR command is received – any of {clear:n}, {“clear”:n}, $clear – the machine will once again act on commands following the CLEAR.
This is complicated by dual endpoint USB, as a CLEAR can be sent via the control channel while there are still commands in the data channel. It is the host’s responsibility to ensure that the data buffers are drained or stopped before CLEAR is sent. (Do we want this to only accept CLEAR on the data channel?)

###Alarm States
The following processing is based on the use cases above.
####Soft_Alarm
A soft alarm is an alarm state where the job is potentially recoverable. The desired behavior is to decelerate (stop) all motion and leave the system open to inspection – preserving as much state as possible.
Soft alarm is triggered in the following cases:
•	A soft limit has been tripped due to a Gcode block exceeding travel limits
•	An input is fired which puts the machine into soft alarm (a flag is set and a function is called)
•	When a queue flush is processed when MOTION_HOLD && FEEDHOLD_HOLD && !runtime_busy() 
(This is current behavior, and should probably be removed.)

Desired processing of soft alarms is outlined by the Limit Switch Hit / Recoverable use case. Current g2 processing of soft alarms is listed below. Some of this may need to change:
•	Machine state is set to MACHINE_ALARM
•	An exception report is returned indicating the machine has entered an Alarm state
•	Commands are rejected in the controller(). This could be a problem as CLEARS won’t get through.
•	Set commands are rejected in the JSON and text parsers (_json_parser_kernal(), text_parser())
•	Commands are rejected in the Gcode parser. This is redundant if SETs are  (redundant with above)
•	Note to self: review exec_program_finalize() behavior relative to alarms.
####Hard Alarm
A hard alarm is an alarm state that is not recoverable. The desired behavior is to stop all motion as fast as possible and leave the system open to inspection – preserving as much state as possible even is position is lost.
Hard alarms are triggered in the following cases:
•	An input that is configured to trigger a hard alarm is fired (e.g. a limit switch)
•	An unrecoverable internal state is detected in a function
•	Assertion failure is detected
Desired processing of hard alarms is outlined by the Limit Switch Hit / Unrecoverable use case. Current g2 processing of soft alarms is listed below. Some of this may need to change:
•	All motion is stopped
•	Machine state is set to MACHINE_SHUTDOWN
•	An exception report is returned indicating the machine has entered a Shutdown state (hard alarm)
•	No further Set commands or Gcode are accepted
•	Get commands are accepted and processed – if possible
•	Hard alarms cannot be cleared. Clears are ignored
####Interlock
See Interlock use case
####eStop
See EStop use case
