_DEV PAGE: This page is a discussion about implementation._
_This is not intended to be end-user data, and G2 may or may not implement what's discussed there._

## There are a few main cases that the board needs to handle:

1. **Stop a Single Move** - Feedhold + queue flush (!%) is used to terminate a single move in cases such as homing, probing, and some forms of jogging. A !% will transition the system to a STOP. At this point the controlling program in the firmware, or on the host can perform the next action.

1. **Stop Multiple Moves** - In some jogging implementations multiple moves may be queued up as part of a single jogging move. In this case the We need the ability to do a !% and take the system to a STOP, while draining all other moves that are queued before the %. It is possible that the sender has filled the planner buffer, serial input RX buffer, USB internal buffers, and even host-based buffers for this single jog move. All these moves need to be flushed before transitioning to a stop.

1. **Kill Job** - Terminate the rest of the job and indicate that all new commands are to be ignored until some point defined by the host. This needs to be able to drain all queued buffers, potentially as far back as the host without having the tool do anything unpredictable (there could be multiple megabytes of data backed up on the host side). This is complicated by needing to work for both single endpoint and dual endpoints (separate control and data channels). Some cases:
  1. Single USB endpoint - gcode commands and ! and % chars share the same channel. ! and % "jump the queue" once they are received by the board.
  1. Dual USB endpoints - one channel is control (! and %), the other is data (gcode). We need to have some way of draining the queued buffers, potentially as far back as the host without having the tool do anything unpredictable. There could be multiple megabytes of data backed up on the host side.
  1. For an SD-card data channel, the solution may be as easy as "close the file". In fact, that would be the behavior we should emulate with the dual-usb.

1. **Reset System** - Do a hard reset if the controller is unresponsive or has entered a SHUTDOWN state.

### Further Background

Additional considerations:

1. The % character is commonly used at the beginning and end of a gcode file to delimit the file.

  * We do NOT plan on supporting running two Gcode files concatenated. Or, at least, we don't plan on adding any special handling for that case.

  * We don't need any special handling in these cases. M2 and M30 handle the "end-of-job" case, and should be used for that purpose.

2. The % character is used by some gcode generators (InkScape, for example) as a start-comment character.

3. Using % as a "kill-job" doesn't make any sense without a ! "feed hold" proceeding it.

### Implementation:

1. **% Handling** [handles cases 1 and 2]: Implement the following behaviors
  1. Intercept % in the serial stream and act on immediately. ]
     1. If the system is in FEEDHOLD (or has one requested) this sets a flag to request queue flush
     1. If the system is in ALARM transition to STOP (this is an auto-clear)
     1. Otherwise the % is ignored
  1. Replace the % with a ; in the serial stream. This allows the % to act as a start-comment character for Gcode comments (supporting the Inkscape comment case), and prevents a comment start from masquerading as an autoclear.

1. **Control-d** [handles case (3) - Kill Job]: Add a new control character end-of-transmission / control-d / ^d. This will set an ALARM state which stops motion and spindle, clears internal planner queues and rejects all queued and incoming commands until a {clear:n} (or $clear) is received. ^d is intercepted by the serial system and is NOT queued, so it is acted on immediately on receipt.

1. **Control-x** [handles case (4) - Kill Job]: Resets the board, exiting a SHUTDOWN state. A shutdown is unrecoverable and requires a reset.

### Notes about V9 serial processing

The current V9 USB serial port implementation relies on the USB hardware's buffers, and only reads data from those buffers as the system can parse them. This doesn't allow for immediate-processing of special characters `!%~` nor does it adhere to the byte-counting flow-control methods. Programs written to use the byte-counting of V8 and that only use a single channel will likely see a "lock up" where the single channel no longer responds to incoming data if only once channel is opened.

In order to solve this, we will add an additional serial buffer to the USB Serial on V9, mimicking the behavior of the V8.
