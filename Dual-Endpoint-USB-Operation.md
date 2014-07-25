_NOTE: THis page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. The basic idea is to treat one as a **data** channel through which the Gcode file is queued, while leaving a second **command** channel open (non-queued) to process command in near-real time. This greatly simplifies flow control for the UI or host, as the data channel can be viewed as always being queued, and the command channel is always available to process incoming commands such as feedholds (stops).

##Theory Of Operation

##Requirements & Use Cases

##Design and Implementation Notes