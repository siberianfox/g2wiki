_NOTE: Functions on this page are in effect as of build 083.xx and later._

See also: [Alarm Processing](Alarm-Processing)

## Summary
Character | Operation | Description
------|------------|---------
! | Feedhold | Start a feedhold. Ignored if already in a feedhold
~ | End Feedhold | Resume from feedhold. Ignored if not in feedhold
% | Queue Flush | Flush remaining moves during feedhold. Ignored if not in feedhold
^d | Kill Job | Trigger ALARM to kill current job. Send {clear:n}, M2 or M30 to end ALARM state
^x | Reset Board | Perform hardware reset to restart the board

## There are a few key use cases that need to be handled:

1. **Stop a Single Move** - Feedhold + queue flush `!%` is used to terminate a single move in cases such as homing, probing, and some forms of jogging. A `!%` will transition the system to a STOP. At this point the controlling program in the firmware, or on the host can perform the next action. (See [g2 differences](Job-Exception-Handling#queue-flush-differences))

1. **Stop Multiple Moves** - In some jogging implementations multiple moves may be queued up as part of a single jogging move. In this case a `!%` needs to take the system to a STOP, but not before draining all other moves that are queued before the `%`. It is possible that the sender has filled the planner buffer, serial input RX buffer, USB internal buffers, and even host-based buffers for this single jog move. All these moves need to be flushed before transitioning to a STOP.

1. **Kill Job** - Terminate the rest of the job and indicate that all new commands are to be ignored until some point defined by the host. This needs to be able to drain all queued buffers, potentially as far back as the host without having the tool do anything unpredictable (there could be multiple megabytes of data backed up on the host side). This is complicated by needing to work for both single endpoint and dual endpoints (separate control and data channels). Some cases:
  1. Single USB endpoint - gcode commands and ! and % chars share the same channel. ! and % "jump the queue" once they are received by the board.
  1. Dual USB endpoints - one channel is control (! and %), the other is data (gcode). We need to have some way of draining the queued buffers, potentially as far back as the host without having the tool do anything unpredictable. There could be multiple megabytes of data backed up on the host side.
  1. For an SD-card data channel, the solution may be as easy as "close the file". In fact, that would be the behavior we should emulate with the dual-usb.

1. **Reset System** - Do a hard reset if the controller is unresponsive or has entered a SHUTDOWN state.

**Additional considerations:**

1. The % character is commonly used at the beginning and end of a gcode file to delimit the file.

  * We don't need any special handling in these cases. M2 and M30 handle the "end-of-job" case, and should be used for that purpose.
  * We do NOT plan on supporting running two Gcode files concatenated. Or, at least, we don't plan on adding any special handling for that case.


2. The % character is used by some gcode generators (InkScape, for example) as a start-comment character.

3. Using % as a "buffer flush" doesn't make any sense without a ! "feed hold" proceeding it.

### Implementation:

1. **% Handling** [cases 1 and 2]: Implement the following behaviors
If the system is not in a feedhold replace the `%` with a `;` in the serial stream. This allows the `%` to act as a start-comment character for Gcode comments (supporting the Inkscape comment case). _Note that g2 treats `;` as a start-comment character even though this is not part of the Gcode specification._

1. **Control-d** [case (3) - Kill Job]: Add a new control character end-of-transmission / control-d / ^d. This will set an ALARM state which stops motion and spindle, clears internal planner queues and rejects all queued and incoming commands until one of the following commands is received. ^d is intercepted by the serial system and is NOT queued, so it is acted on immediately on receipt.
  * {clear:n}
  * $clear
  * M2
  * M30

1. **Control-x** [case (4) - Reset System]: Resets the board, exiting a SHUTDOWN state. A shutdown is unrecoverable and requires a reset:
  * `^x`
  * hardware reset
  * power cycle


### Notes about g2 serial processing
#### No Byte Counting
The current g2 USB serial port implementation relies on the USB hardware's buffers, and only reads data from those buffers as the system can parse them. This doesn't allow for byte-counting flow-control methods. Programs written to use the byte-counting of v8 and that only use a single channel will likely see a "lock up" where the single channel no longer responds to incoming data if only once channel is opened.

In order to solve this, we will add a line-mode (packet-mode) to the USB Serial on g2, allowing the host to send one line at a time and only need to keep track of the number of available lines in the line buffer (no character counting)

#### Special Characters
the g2 USB serial does not perform immediate-processing of special characters `!`, `%`, `~`, `^x`, or `^d`. Instead it receives these from the USB sub-system and executes them as part of the normal command dispatch. However, these characters do take priority over a single command, and can "jump the queue", but will not jump the queue over multiple commands that may be backed up in the on-board USB buffers. The line-mode processing should also take care of this issue.

#### Queue Flush Differences
In g2 the queue flush operation (%) ends a feed hold if one is in effect. The `!%~` sequence can be shortened to `!%` The `~` is wiped from the serial RX buffer and not executed independently.
