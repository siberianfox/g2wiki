##Machine State and Alarm Processing
Related pages
- [Digital IO (GPIO)](Digital-IO-(GPIO))
- [Job Exception Handling](Job-Exception-Handling)

###Machine States
Machine states are listed below. Interlock and panic are new additions to the combined machine state. The behavior of some other states have been slightly redefined, as described in the remainder of this page. 

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
	COMBINED_CYCLE,         // [8] reserved for canned cycles
	COMBINED_HOMING,        // [9] homing cycle 
	COMBINED_JOG,           // [10] jogging cycle active
	COMBINED_INTERLOCK,     // [11] machine in safety interlock hold
	COMBINED_SHUTDOWN,      // [12] machine in shutdown state
	COMBINED_PANIC          // [13] machine in panic state
};
</pre>

##Description of Alarm Exception States
###ALARM
Alarm is typically entered by a soft limit, a limit switch, or a safety interlock being disengaged. The following occur:

- Set ALARM machine state
- Start a feedhold to stop motion
- Optionally stop spindle
- Optionally turn off coolant
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands (gcode blocks, SET commands, and other actions) until the alarm is cleared. Non-action commands are still processed (GETs) so the system can be inspected during an alarm. Rejected commands are responded with a 204 status code: STAT_COMMAND_REJECTED_BY_ALARM
- Motor power management remains in effect, although the machining cycle is now over. Motors that are "always on" will remain energized, as these motors may require power to avoid crashing.
- Generate a JSON exception report indicating the ALARM state and the input channel or command line that invoked the alarm

Alarms can be manually cleared by entering: {clear:n}, {clr:n}, $clear, or $clr. Alarms will also clear on receipt of an M30 or M2 command if one is received while draining the host command queue (i.e. when rejecting new commands from the host USB input).

In the case where the alarm is triggered by a limit switch the input line's INPUT_ACTION setting overrides the feedhold - i.e. if the input action is "FAST_STOP" or "HALT" that setting will take precedence over the feedhold native to the alarm function.

Job state and and machine state is preserved and will report the position at the end of the feedhold. Since ALARM attempts to preserve Gcode and machine state it does not END the job unless an explicit M30 or M2 is received. It may be possible to recover a job from an alarm, but in many cases this is not possible.

An ALARM may also be invoked from the command line sending {alarm:n} or $alarm 

###SHUTDOWN
Shutdown is typically invoked as an electrical input signal sent to the board as
part of an external emergency stop (Estop).  Shutdown is meant to augment but not
replace external Estop functions that shut down power to motors, spindle, coolant and
other moving parts. The following actions occur:

- Set SHUTDOWN machine state
- Stop all motion immediately - motors HALT without deceleration
- Stop spindle (unconditionally)
- Turn off coolant (unconditionally)
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands (gcode blocks, SET commands, and other actions) until the shutdown is cleared. Non-action commands are still processed (GETs) so the system can be inspected during a shutdown. Rejected commands are responded with a 205 status code: STAT_COMMAND_REJECTED_BY_SHUTDOWN
- Motor power management remains in effect, although the machining cycle is now over. Motors that are "always on" will remain energized, as these motors may require power to avoid crashing. It is the responsibility of the external Estop system to determine is power is still applied to these motors.
- Generate a JSON exception report indicating the SHUTDOWN state and the input channel or command line that invoked the shutdown

Shutdown must be manually cleared by entering: {clear:n}, {clr:n}, $clear, or $clr, or by a hard or soft system reset (^x). Shutdowns do not clear with an M30 or M2.

Shutdown may also be invoked from the command line by sending {shutd:n} or $shutd

###PANIC
A PANIC occurs if the firmware has detected an unrecoverable internal error such as an assertion failure or a code condition that should never occur. Panics may also be invoked from input switches or command input. The following occur:

- Set PANIC machine state
- Stop all motion immediately - motors HALT without deceleration
- Stop spindle (unconditionally)
- Turn off coolant (unconditionally)
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands. Non-action commands are still honored (GETs), assuming the system is still operational enough to be inspectable. Rejected commands are responded with a 206 status code: STAT_COMMAND_REJECTED_BY_PANIC
- Motor power management remains in effect, although the machining cycle is now over.
- Generate a JSON exception report indicating the PANIC state and the input channel, command line, or coded error that invoked the panic

PANIC can only be exited by a hardware reset or soft reset (^x)

### CLEARs
An ALARM may be cleared by any of:

- {clear:n}, {“clear”:n}, {clr:n}, {“clr”:n} --- JSON options
- $clear, $clr --- Text-mode options
- M30, M2 --- Gcode options
- ^x (control x) --- resets the system

SHUTDOWN may be cleared by any of:

- {clear:n}, {“clear”:n}, {clr:n}, {“clr”:n} --- JSON options
- $clear, $clr --- Text-mode options
- ^x (control x) --- resets the system

PANIC may only be cleared by:

- ^x (control x) --- resets the system

####Clear on Dual Endpoint USB 
Using clear is slightly more complicated on dual endpoint USB. Clears should be sent over the data channel, as this ensures that all queued commands are cleared up to the clear command itself. A clear will be honored if sent via the control channel but this practice is discouraged as it can lead to unpredictable results. Clears received on the control channel will be warned in the response. In this case it is the host’s responsibility to ensure that the data buffers are drained or stopped before CLEAR is sent.

###Other Topics

####Alarm Enables
There are a few settings that enable and disable various alarm cases:

- {sl:1} Soft limit enable: set to 0 or 1
- {lim:1} Limit switch enable: set to 0 or 1
- {saf:1} Safety interlock enable: set to 0 or 1

Setting limit switch enable to 0 is the same as a limit override. This can be used to drive the system off a limit switch. Note that if the system is in an alarm state the alarm must be cleared prior to changing this value.

####Spindle and Coolant Optional Behavior
The spindle and coolant can be instructed to pause on feedhold by setting:

- {sph:0}, $sph=0 --- spindle pause on hold disabled
- {sph:1}, $sph=1 --- spindle pause on hold enabled

- {coph:0}, $coph=0 --- coolant pause on hold disabled
- {coph:1}, $coph=1 --- coolant pause on hold enabled
 
The pause will resume once the hold is lifted. These options affect any feedhold case, which therefore includes soft limits, limit switches, and safety interlocks.

##Alarm Use Cases
The following use cases are supported:

####Soft Limit
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

####Limit Switches
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

####Safety Interlock Engaged/Disengaged
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

####Emergency Shutdown
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

####Unrecoverable Internal Error - Panic
- **Panic**: An internal condition is detected that indicates controller malfunction of some other unrecoverable problem. Internal shutdown is handled identically to emergency shutdown, with the exception of the contents of the exception report. In some cases an exception report cannot be generated. If you see this case please note the exception report contents and let us know about it.  
