##Machine State and Alarm Processing
Related pages
- [Digital IO (GPIO)](Digital-IO-(GPIO))

###Machine States
Machine states are listed below. Interlock is a new addition to the combined machine state. The behavior of some other states have been slightly redefined, as described in the remainder of this page. 

<pre>
enum cmCombinedState {
	COMBINED_INITIALIZING,  // [0] machine is initializing
	COMBINED_READY,         // [1] machine is ready for use
	COMBINED_ALARM,         // [2] machine in alarm state (soft or hard)
	COMBINED_PROGRAM_STOP,  // [3] program stop/no more blocks
	COMBINED_PROGRAM_END,   // [4] program end
	COMBINED_RUN,           // [5] motion is running
	COMBINED_HOLD,          // [6] motion is holding
	COMBINED_PROBE,         // [7] probe cycle active
	COMBINED_CYCLE,         // [8] DEPRECATED: now just COMBINED_RUN
	COMBINED_HOMING,        // [9] homing cycle 
	COMBINED_JOG,           // [10] jogging cycle active
	COMBINED_SHUTDOWN,      // [11] machine in shutdown (due to emergency stop)
	COMBINED_INTERLOCK      // [12] machine in interlock hold
};
</pre>

####Alarm Enables
There are a few settings that enable and disable various alarm cases:

- {sl:1} Soft limit enable: set to 0 or 1
- {lim:1} Limit switch enable: set to 0 or 1
- {ilck:1} Interlock enable: set to 0 or 1

Setting limit switch enable to 0 is the same as a limit override. This can be used to drive the system off a limit switch. Note that if the system is in an alarm state the alarm must be cleared prior to changing this value using {clear:n} or $clear

###Alarm Use Cases
The following use cases are supported:

- **Soft Limit**: A Gcode block is received that would exceed the maximum or minimum travel in one or more axes. This could be true because the cutting path exceeds the available machine extents, or because the part was located (zeroed) on the table incorrectly (e.g. “not centered”). In either case assume the job is not recoverable. Soft limits are only applied if the machine is homed. The desired behavior is triggered as soon as the Gcode block is interpreted, and is:
  - Transition to ALARM state (from RUN)
  - Stop movement while preserving position (feedhold to a STOP)
    - If already in feedhold do not execute the STOP
  - Stop spindle or other actuator (extruder, laser, torch…)
  - No change to coolant output
  - Motor power timeouts activate based on motor power settings – e.g. only-when-moving or in-cycle
  - Flush the planner queue of any remaining moves
  - Drain the serial queues: Reject any motion commands and parameter setting commands.
  - Generate exception report for SOFT_LIMIT w/line number of the limit (if Gcode has line numbers)
  - Transition to PROGRAM_END state on receipt of CLEAR command

- **Limit Switch Hit**: A limit switch has been hit. Depending on the action settings, machine type, velocities, switches and other factors the position may or may not be preserved. The desired behavior is:
  - Transition to ALARM state (from RUN)
  - Stop movement with our without preserving position (STOP, FAST_STOP, HALT)
    - Do not perform this action already started a feedhold
  - Stop spindle or other actuator (extruder, laser, torch…)
  - No change to coolant output
  - Motor power timeouts activate based on motor power settings – i.e. no longer moving or in cycle
  - Flush the planner queue of any remaining moves
  - Drain the serial queues: Read and reject motion and SET commands (See CLEAR command).
  - Mark the limit axis as UNHOMED and mark the machine as UNHOMED
  - Generate exception report for LIMIT HIT, and indicate which input was tripped
  - Transition to PROGRAM_END state on receipt of CLEAR command 
  - Accept {lim:0} (LIMIT_OVERRIDE) to disable limits before moving machine off limit switch
  - Host should reset {lim:1} before the next cycle

- **Interlock Hold**: An interlock condition has been detected on an input. Interlocks should allow jobs to resume once the interlock condition has been removed. The desired behavior is:
  - Transition to INTERLOCK state (from RUN)
  - Stop movement with rapid deceleration while preserving position (HALT)
    - If already in feedhold do not execute the HALT
  - Stop spindle or other actuator (extruder, laser, torch…)
  - No change to coolant output
  - Motor power timeouts activate based on motor power settings – i.e. no longer moving or in cycle
    - Q: Or do we want them to stay energized indefinitely?
  - Generate exception report for entering INTERLOCK condition w/the interlock input number
  - Wait for interlock condition to be cleared by the input
  - Generate exception report for exiting INTERLOCK condition w/the interlock input number
  - Restore spindle to prior state, with delay dwell of N seconds (settable)
  - Resume motion by resuming from feedhold (CYCLE_START)
    - ...unless already in feedhold at the time of INTERLOCK

- **Emergency Shutdown**: Emergency shutdown is the controller’s reaction to an external Emergency Stop (EStop) being activated. Note that Estop is NOT a controller function, it is an external condition that must remove power and/or brake all moving parts, including axes, spindles, etc. This occurs outside of the controller’s domain, as Estop needs to function even if the controller has malfunctioned. The Emergency Shutdown input is used to tell the controller that an Estop condition has occurred, and for it to take appropriate steps on its own. Emergency Shutdown is not recoverable. The desired behavior is:
  - Transition to SHUTDOWN state (from RUN)
  - Stop movement immediately without preserving position (HALT)
  - Stop spindle or other actuator (extruder, laser, torch…)
  - Stop coolant output
  - Disable motor power (no timeout)
  - Flush the planner queue
  - Mark all axes as UNHOMED and mark the machine as UNHOMED
  - Drain the queues: Read and reject motion and SET commands (See CLEAR command).
  - Exception report is generated reporting SHUTDOWN condition
  - Shutdown is only recoverable by a RESET (soft or hard), or power cycle
    - Note: The external Estop logic or the host may decide to reset the controller

- **Internal Shutdown**: An internal condition is detected that indicates controller malfunction of some other unrecoverable problem. Internal shutdown is handled identically to emergency shutdown, with the exception of the contents of the exception report. In some cases an exception report cannot be generated. If you see this case please note the exception report contents and let us know about it.  

### CLEAR Command
When entering an ALARM state there may be Gcode and configuration commands buffered in multiple places including the planner buffer, on-board serial receive queues (RX), onboard USB queues, various sender and other queues on the host.
 
In an alarm it is desirable to cease all motion, so this requires rejecting all new commands that may be received after the alarm condition is triggered. Commands received in an state will be read but not processed, and will return with status code 203, STAT_MACHINE_ALARMED.

In this state it is possible to query the board, (i.e. GET commands), but not possible to change the state of the machine (i.e. SET commands or Gcode)

Once a CLEAR command is received – any of {clear:n}, {“clear”:n}, $clear – the machine will once again act on commands following the CLEAR.

####Clear on Dual Endpoint USB 
Using clear is more complicated on dual endpoint USB. Clears should be sent over the data channel, as this ensures that all commands are cleared up to the clear command itself. Clear can be sent via the control channel but this practice is discouraged as it can lead to unpredictable results. Clears received on the control channel will be honored, but will be warned in the response. In this case it is the host’s responsibility to ensure that the data buffers are drained or stopped before CLEAR is sent.
