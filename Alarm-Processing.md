##Machine State and Alarm Processing
This page describes the internal machine state and how exception processing such as alarms and shutdown affect this state.

Related pages
- [Digital IO (GPIO)](Digital-IO)
- [Job Exception Handling](Job-Exception-Handling)

### Machine States
Machine states are listed below. Interlock and panic are new additions to the combined machine state. The behavior of some other states have been slightly redefined, as described in the remainder of this page.

<pre>
enum cmCombinedState {
	COMBINED_INITIALIZING,  // [0] machine is initializing
	COMBINED_READY,         // [1] machine is ready for use
	COMBINED_ALARM,         // [2] machine in alarm state
	COMBINED_PROGRAM_STOP,  // [3] program stop/no more blocks
	COMBINED_PROGRAM_END,   // [4] program end
	COMBINED_RUN,           // [5] machine is running
	COMBINED_HOLD,          // [6] machine is holding
	COMBINED_PROBE,         // [7] probe cycle active
	COMBINED_CYCLE,         // [8] reserved for canned cycles
	COMBINED_HOMING,        // [9] homing cycle active
	COMBINED_JOG,           // [10] jogging cycle active
	COMBINED_INTERLOCK,     // [11] machine in safety interlock hold
	COMBINED_SHUTDOWN,      // [12] machine in shutdown state
	COMBINED_PANIC          // [13] machine in panic state
};
</pre>

## Description of Alarm Exception States
These exception states support the [alarm use cases](#alarm-use-cases) later on this page.
### ALARM
Alarm is typically entered by a soft limit, a limit switch, or a safety interlock being disengaged. The following occur:
- Set ALARM machine state
- Start a feedhold to stop motion
- Optionally stop spindle
- Optionally turn off coolant
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands (gcode blocks, SET commands, and other actions) until the alarm is cleared. Non-action commands are still processed (GETs) so the system can be inspected during an alarm. Rejected commands are responded with a 204 status code: STAT_COMMAND_REJECTED_BY_ALARM
- Motor power management remains in effect, although the machining cycle is now over. Motors that are "always on" will remain energized, as these motors may require power to avoid crashing.
- Generate a JSON exception report indicating that an ALARM has occurred and the input channel, command line, or other source of the alarm

Alarms can be manually cleared by entering {clear:n} or a [variant](#clear). Alarms will also clear on receipt of an M30 or M2 command if one is received while draining the host command queue (i.e. when rejecting new commands from the host USB input).

Job state and and machine state is preserved and will report the position at the end of the feedhold. Since ALARM attempts to preserve Gcode and machine state it does not END the job unless an explicit M30 or M2 is received. It may be possible to recover a job from an alarm, but in many cases this is not possible.

In the case where the alarm is triggered by a limit switch the INPUT_ACTION setting may override the feedhold - i.e. if the input action is "FAST_STOP" or "HALT" that setting will take precedence over the feedhold native to the alarm function. The limit input can also be set up to PANIC or RESET, if so desired.

An ALARM may also be invoked from the command line sending {alarm:n} or $alarm

### SHUTDOWN
Shutdown is typically invoked as an electrical input signal sent to the board as part of an external emergency stop (Estop).  Shutdown is meant to augment but not replace external Estop functions that shut down power to motors, spindle, coolant and other moving parts. The following actions occur:
- Set SHUTDOWN machine state
- Stop all motion immediately - motors HALT without deceleration
- Stop spindle (unconditionally)
- Turn off coolant (unconditionally)
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands (gcode blocks, SET commands, and other actions) until SHUTDOWN is cleared. Non-action commands are still processed (GETs) so the system can be inspected during shutdown. Rejected commands are responded with a 205 status code: STAT_COMMAND_REJECTED_BY_SHUTDOWN
- Motor power management remains in effect, although the machining cycle is now over. Motors that are "always on" will remain energized, as these motors may require power to avoid crashing. It is the responsibility of the external Estop system to determine is power is still applied to these motors.
- Generate a JSON exception report indicating the SHUTDOWN state and the input channel or command line that invoked the shutdown
- Generate a JSON exception report indicating that a SHUTDOWN has occurred and the input channel or command line source of the shutdown

Shutdown must be manually cleared by entering: {clear:n} or a [variant](#clear), or by a hard or soft system reset (^x). Shutdowns do not clear with an M30 or M2.

Shutdown may also be invoked from the command line by sending {shutd:n} or $shutd

### PANIC
A PANIC occurs if the firmware has detected an unrecoverable internal error such as an assertion failure or a code condition that should never occur. Panics may also be invoked from input switches or command input. The following occur:

- Set PANIC machine state
- Stop all motion immediately - motors HALT without deceleration
- Stop spindle (unconditionally)
- Turn off coolant (unconditionally)
- Flush queued planner moves and any commands in the serial buffer
- Reject new action commands. Non-action commands are still honored (GETs), assuming the system is still operational enough to be inspectable. Rejected commands are responded with a 206 status code: STAT_COMMAND_REJECTED_BY_PANIC
- Motor power management remains in effect, although the machining cycle is now over.
- Generate a JSON exception report indicating that a PANIC has occurred and the input channel, command line, or coded error that invoked the panic

PANIC can only be cleared by a hardware reset or soft reset (^x)

### CLEAR
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

####C lear on Dual Endpoint USB
Using clear is slightly more complicated on dual endpoint USB. Clears should be sent over the data channel, as this ensures that all queued commands are cleared up to the clear command itself. A clear will be honored if sent via the control channel but this practice is discouraged as it can lead to unpredictable results. Clears received on the control channel will be warned in the response. In this case it is the host’s responsibility to ensure that the data buffers are drained or stopped before CLEAR is sent.

### Other Topics

#### Alarm Source Enables
These settings enable and disable various alarm sources:

- {sl:1} Soft limit enable: set to 0 or 1
- {lim:1} Limit switch enable: set to 0 or 1
- {saf:1} Safety interlock enable: set to 0 or 1

Setting limit switch enable to 0 is the same as a limit override. This can be used to drive the system off a limit switch. Note that if the system is in an alarm state the alarm must be cleared prior to changing this value.

#### Spindle and Coolant Optional Behavior
The spindle and coolant can be instructed to pause during a feedhold by setting:

- {sph:0} Spindle pause-on-hold disabled
- {sph:1} Spindle pause-on-hold enabled
- {coph:0} Coolant pause-on-hold disabled
- {coph:1} Coolant pause-on-hold enabled

(...or their text-mode equivalents). The pause will resume once the hold is lifted. These options affect any feedhold case, which therefore includes soft limits, limit switches, and safety interlocks.

## Alarm Use Cases
Alarm state handling supports the following use cases, some of which are implemented "on top of" the alarm processing.

#### Soft Limit
A Gcode block is received that would exceed the maximum or minimum travel in one or more axes. This could be true because the cutting path exceeds the available machine extents, or because the part was located (zeroed) on the table incorrectly (e.g. “not centered”). In either case assume the job is not recoverable. Soft limits are only applied if the machine is homed. The desired behavior is triggered as soon as the Gcode block is interpreted.
- Run [ALARM processing](#alarm), stopping movement at the time the errant Gcode line is received - i.e. not later, when it was to be executed
- Soft limits always use normal feedholds, never fast_stop or halt

#### Limit Switches
A limit switch has been hit. Depending on the action settings, machine type, velocities, switches and other factors the position may or may not be preserved. The desired behavior is:
- Run [ALARM processing](#alarm) (the INPUT ACTION may override the default feedhold with a faster stop)
- Mark the affected axis as UNHOMED and mark the machine as UNHOMED
- Once the alarm is cleared, accept {lim:0} (LIMIT_OVERRIDE) to disable limits before moving machine off the limit switch
- Host should reset {lim:1} before starting the next cycle

#### Safety Interlock Engaged/Disengaged
A safety interlock switch or detector has disengaged. Motion should stop. The job should to resume once the interlock is re-engaged. The desired behavior is:
- Run [ALARM processing](#alarm)
- Transition to INTERLOCK state
- Generate exception report indicating INTERLOCK disengaged (includes the interlock input number)
- Wait for interlock condition to be cleared by the input
- Generate exception report indicating INTERLOCK re-engaged (includes the interlock input number)
- Restore spindle to prior state, with delay dwell of N seconds (settable)
- Resume motion by resuming from feedhold (CYCLE_START)
  - ...unless already in feedhold at the time of INTERLOCK

Note: None of the above will occur if the (saf:0}. If saf is set to 1 (enabled) during an INTERLOCK state the motion should resume as if the safety interlock had been re-engaged by its switch.

#### Emergency Shutdown
Emergency shutdown is the controller’s reaction to an external Emergency Stop (EStop) being activated. Note that Estop is NOT a controller function. Estop is an external condition that must remove power and/or brake all moving parts, including axes, spindles, etc. This occurs outside of the controller’s domain, as Estop needs to work even if the controller has malfunctioned. One or more inputs configured for SHUTDOWN are used to tell the controller that an Estop has occurred, and for the controller to take appropriate steps. Jobs in process during SHUTDOWN are not recoverable. The desired behavior is:
- Run [SHUTDOWN processing](#shutdown)
- Mark all axes as UNHOMED and mark the machine as UNHOMED
- Generate exception report reporting SHUTDOWN condition and shutdown source
- Shutdown is recoverable by a clear, a hard or soft reset, or a power cycle
  - Note: The external Estop logic or the host may decide to reset the controller

#### Unrecoverable Internal Error - Panic
An internal condition is detected that indicates controller malfunction of some other unrecoverable problem. Internal shutdown is handled by [PANIC](#panic).
