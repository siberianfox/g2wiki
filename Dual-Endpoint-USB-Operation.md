_NOTE: This page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first is a **data** channel through which the Gcode file is queued. This channel is expected to have a significant number of Gcode lines (blocks) queued up, ready for execution. The second channel is a **command** channel. This is always open to process command in near-real time. This greatly simplifies flow control for the UI or host, as the data channel can be viewed as always being queued, and the command channel is always available to process incoming commands such as feedholds (stops).

##Requirements & Use Cases

##Design and Implementation Notes