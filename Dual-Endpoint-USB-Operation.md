_NOTE: This page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first is a **data** channel through which the Gcode file is queued. This channel is expected to have a significant number of Gcode lines (blocks) queued up, ready for execution. The second channel is a **control** channel. This is always open to process commands in near-real time.

This arrangement greatly simplifies flow control for the UI or host, as the data channel can be viewed as always being queued (blocked), and the control channel can be viewed as always being available to process incoming commands such as feedholds (stops).

##Requirements & Use Cases

* UC_1: Two USB serial ports - The USB has two virtual serial ports configured.
 
* UC_2: One USB Serial Port, One mass storage device - One virtual serial port is configured for the control channel. Data is made available from a mass storage device. Sub-cases include data channel being from (a) USB mass storage device, (b) SD card or similar off-board, serial accessed storage, (c) on-chip FLASH or EEPROM. Each case may assume read/write or read only mass storage.

* UC_3: One USB serial port - Both USB serial ports are "piggybacked" on the same channel. Used to support v8 and earlier boards and cases where 2 endpoints are not available for some reason.

##Functional Specifications
Let's suppose it works like this:

**Communications**<br>
The following are expected on the data channel and the control channel.
* Data Channel
  * Raw Gcode blocks or JSON wrapped Gcode blocks are presented on the data channel
  * Blocks are read one line at a time
  * Lines terminated with LF, CR or both
  * All input is interpreted as Gcode (or whatever data prototocl is in effect). All else is treated as an error.
  * No response is provided back on the data channel (no echo, acknowledgements, or errors)
* Control Channel
  * Control channel accepts all other commands and controls, including:
    * Configuration commands (JSON and text mode)
    * Single-character controls: !, % ~
  * Control channel returns all responses, including
    * JSON response objects
    * JSON status reports
    * JSON exception reports
    * text-mode responses of all kinds
    * Gcode comment messages

**Channel Assignment**<br>
Initially neither channel is assigned as data or control. 
* The first channel to receive any of these characters will become the control channel:
  * {
  * ?
  * $
  * !
  * %
  * ~
* When initially assigned the control channel will accept control and data (UC_3)
* To assign the other channel as a data channel send `{data:"second"}`
* To disconnect and return to an un-assigned state send three ESC characters in a row. (0x1B)

**Assign Data Channel to and Alternate Source (UC_2)**
* To assign the data channel to an alternate source send: `{data:"source"}` to the command channel. where _source_ is a device name or file name
* If the data source closes due to end-of-file or other error condition the data channel reverts to the default data channel
* To force the data channel back issue `{data:"default"}` to the command channel.

**Single Channel Mode (UC_3)**


##Design and Implementation Notes