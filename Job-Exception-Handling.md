# DEV PAGE: This page is a discussion about future implementation.

_This is not intended to be end-user data, and G2 may or may not implement what's discussed there._

## We have two cases that we need to discuss and come up with solutions to:

1. **Job Failure** - while  a job is running under a dual-channel system (two USB, a USB control and a SD data, etc), we need a way for the control channel to abort the rest of the job and indicate that everything on the data channel is to be ignored until some point.
  1. For an SD-card data channel, the solution may be as easy as "close the file". In fact, that would be the behavior we should emulate with the dual-usb.
  1. For Dual USB, we need to have some way of saying "clear until here" in the data channel, and the system should report back when that's completed, since there could be multiple megabytes of data backed up on the host side.
1. **Don't Complete Motion** - For cases such as manual entry of commands or jogging, we need the ability to do a feed-hold-and-queue-flush.
  1. In the jogging case, we may actually want the next command in the serial stream maintained. (???)
  1. Perhaps we should make them an atomic action?

Also, our queue-flush character (%) is also a start-of-file marker output by some gcode generators, and should be ignored.

## Implementation Discussion

We have two separate use cases. Are they demanding separate implementations, or can we solve them both at the same time?

## Queue Flush

There are a few ways this could be done.  One would be to wait for a g-code "End of program" (M2, M30) on the G-code port once a queue flush is received on the control port, and use that as a marker to allow for the release of stuff backed up on the host.

Another could be to just read everything on the g-code port when a queue flush is received until a timeout occurred.  Presumably, when a flush is recieved, the host is no longer actively sending data, so if you're worried about anything, you're only worried about the stuff that's sitting in the buffer on the host.  If you just started reading in a loop, and bailed once you had a read timeout of a second or so, you could be pretty sure that you got everything that was backed up on the host.

The second method above carries the disadvantage of costing a second every time you do a queue flush, which would be extremely undesirable in many cases.  The first solution is more of a "closed loop" - it essentially requires the sending of a token to indicate the end of the buffer on the G-Code port.  This of course carries the new possible error of having received a feed hold on the control port, but never actually seeing the end of program (for whatever reason) on the G-code port.

Another way of dealing with this is potentially just leave it up to the PC to flush its output buffer? As I've said above, the PC should be clever enough not to send down any more data once it's requested a queue flush, and this could extend to its output buffer as well.  I guess the correct chain of events on the host side is something like this:

 * Need to stop the tool is encountered
 * issue feed hold
 * Stop sending g-codes on g-code port
 * Flush g-code port (drain)
 * issue queue flush

## Implementation suggestion:

### Background

Cases to be handled:

1. The % character is used at the beginning and end of a gcode file traditionally in order to delimit the file.

  * We do NOT plan on supporting running two files concatenated. Or, at least, we don't plan on adding any special handling for that case.

  * We don't need any special handling in these cases. M2 and M30 handle the "end-of-job" case, and should be used for that purpose.

2. The % character is used by some gcode generators (InkScape, for example) as a beginning-of-comment.

3. Using % as a "kill-job" doesn't make any sense without a ! "feed hold" proceeding it.

### Implementation:

1. If we *are* in a feed-hold or in the process of a feed-hold, we will:

  1. Put the machine into soft-alarm. This board will continue to read from both data and control channels and will consume but ignore any new (queued) data. New lines will not be executed (they're tossed) and are responded with a 203 MACHINE_ALARMED status. The host will need to send a `{clr:n}` to clear the soft alarm before any further processing can occur. The clear may be sent on the data or combined (data+control) channel. As the board will continue to consume commands and respond with 203 this has the effect of draining the local serial buffers and potentially any data queued at the host end.

  1. ...and flush the planner queue, effectively resetting all motion.


2. If we are *not* in a feed-hold:

  1. We will treat the % as a comment character, and ignore the rest of the line that it was on. There will be no further special handling of %.

### Notes about V9 serial processing

The current V9 USB serial port implementation relies on the USB hardware's buffers, and only reads data from those buffers as the system can parse them. This doesn't allow for immediate-processing of special characters `!%~` nor does it adhere to the byte-counting flow-control methods. Programs written to use the byte-counting of V8 and that only use a single channel will likely see a "lock up" where the single channel no longer responds to incoming data if only once channel is opened.

In order to solve this, we will add an additional serial buffer to the USB Serial on V9, mimicking the behavior of the V8.
