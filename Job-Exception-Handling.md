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

Cases for handling %

* % received at beginning of Gcode file with no other characters on that line. I.e the machine is not in a machining cycle. Use case: A lone % is common at the start of a Gcode file, by convention. It is meant to "clear" the system.
  * Received over data channel: Flush planner queue. Do not flush serial queue. Permit continued command and data processing
   * Received over data channel: same as above
   * Received over combined channel: same as above

* % at end of Gcode file. Machine is not in cycle, i.e. following M2 or M30. A % is common at the end of a Gcode file, by convention. It is meant to "clear" the system.
  * Received over data channel: Flush planner queue. Do not flush serial queue. Permit continued command and data processing
   * Received over data channel: same as above
   * Received over combined channel: same as above
   * NOTE: It is not expected, nor possible, to stream 2 gcode jobs (files) w/%'s back-to-back. You must quesce the serial system prior to introducing the second job.

* % at end of Gcode file. Is in cycle - i.e. no M2 or M30

* % in middle of Gcode file / cycle masquerading as a comment (the Inkspace case). 

* % after feedhold 

* % in cycle on control channel (v9 only)

