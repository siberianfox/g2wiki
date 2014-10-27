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