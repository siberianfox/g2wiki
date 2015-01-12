# DEV PAGE: This page is a discussion about future implementation.

_This is not intended to be end-user data, and G2 may or may not implement what's discussed there._

## We have two cases that we need to discuss and come up with solutions to:

1. **Job Failure** - while  a job is running under a single or dual-channel system (two USB, a USB control and a SD data, etc) we need a way for the control channel to abort the rest of the job and indicate that everything on the data channel is to be ignored until some point.
  1. For an SD-card data channel, the solution may be as easy as "close the file". In fact, that would be the behavior we should emulate with the dual-usb.
  1. For Dual USB, we need to have some way of draining the queued buffers, potentially as far back as the host without having the tool do anything unpredictable. There could be multiple megabytes of data backed up on the host side.
1. **Don't Complete Motion** - For cases such as manual entry of commands or jogging, we need the ability to do a feed-hold-and-queue-flush.

Also, our queue-flush character (%) is also a start-of-file / end-of-file marker output by some gcode generators, and may be used as a comment character for Inkscape generated files. These cases need to be handled as well.

### Further Background

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
