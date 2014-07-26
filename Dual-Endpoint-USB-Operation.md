_NOTE: This page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first is a **data** channel through which the Gcode file is queued. This channel is expected to have a significant number of Gcode lines (blocks) queued up, ready for execution. The second channel is a **control** channel. This is always open to process commands in near-real time.

This arrangement greatly simplifies flow control for the UI or host, as the data channel can be viewed as always being queued (blocked), and the control channel can be viewed as always being available to process incoming commands such as feedholds (stops).

##Requirements & Use Cases

* UC_1: Two USB serial ports - The USB has two virtual serial ports configured, control and data.
 
* UC_2: One USB Serial Port, One mass storage device - One virtual serial port is configured for the control channel. Data is made available from a mass storage device. Sub-cases include data channel being from (a) USB mass storage device, (b) SD card or similar off-board, serial accessed storage, (c) on-chip FLASH or EEPROM. Each case may assume read/write or read only mass storage.

* UC_3: One USB serial port - Both control and data are "piggybacked" on the same USB port. This mode is used to support v8 and earlier boards and cases where 2 endpoints are not available for some reason.

##Functional Specifications

**Functions**<br>
The following are expected on the data channel and the control channel.
* Data Channel
  * Raw Gcode blocks or JSON wrapped Gcode blocks are presented on the data channel
  * Blocks are read one line at a time
  * Lines terminated with LF, CR or both
  * All input is interpreted as Gcode (or whatever data protocol is in effect). All else is treated as an error. **[This seems to invalidate the ability to have JSON-wrapped Gcode. I suggest removing JSON-wrapped Gcode as valid from this channel. -Rob]**
  * No response is provided back on the data channel (no echo, acknowledgements, or errors)
* Control Channel
  * Control channel accepts all other commands and controls, including:
    * Configuration commands (JSON and text mode)
    * Single-character controls: !, %, ~
  * Control channel returns all responses, including
    * JSON response objects
    * JSON status reports
    * JSON exception reports
    * text-mode responses of all kinds
    * Gcode comment messages

**Channel Assignment**<br>
Initially neither channel is assigned as data or control. 
* The control and data channels are known as:
  * /tty0 is the control channel
  * /tty1 is the data channel
* The first channel to receive any of these characters will become the control channel:
  * {
  * ?
  * $
  * !
  * %
  * ~
* When initially assigned the control channel will accept control and data (UC_3 mode)
* To assign the other channel as a data channel send `{data:"/tty1"}` (UC_1 mode)
* To disconnect and return both channels to an uninitialized state send three ESC characters in a row (0x1B) to any open channel. This works regardless of the use case, UC_1, UC_2 or UC3.

**Assign Data Channel to and Alternate Source (UC_2 mode)**
* To assign the data channel to an alternate source send: `{data:"source"}` to the command channel, where "source" is a device name or file name
* If the data source closes due to end-of-file or other error condition the data channel reverts to the original "dual" data channel **[Does this mean UC_1 mode or UC_3 mode? -Rob]**
* To force the data channel back to dual at any time issue `{data:"/tty1"}` to the command channel


##Design and Implementation Notes

* Some channels will be read/write `RW`, and others will be read-only `R` (plus flow control).
* Of the ones that are `RW` will be a subset that are interactive `I` and capable of sending commands (JSON, etc), receiving status reports, and sending GCode.
* One other situation is where there's an interactive device that cannot transmit GCode, such as a front-panel device. These are interactive command-only `IC` devices. 
  * A USB serial device is default `I`. It will become `RW` when set as a data channel. 
  * An SD card is either `R` or `RW`, but never `I`. (Note: You still talk to an SD card to get data, even when the card is `R`.)
  * An SPI device may be `R`, `RW`, `I`, or `IC`. There needs to be a mechanism to determine which. 
* When an non-`I` device, such as an SD card, is the Data channel, it's expected to automatically keep the gcode buffer full until there is no more (EOF, etc).
* When selecting a Data channel, the rest of the path beyond that necessary to designate the device itself, if any, will be used to determine a subset of the channel.
  * "/sd/file/path" will pass "/file/path" to the sd channel. It might not *accept* the Data channel designation (file not found, card missing, etc).
  * "/spi0/cs1" will pass "/cs1" to the SPI device for determining which sub-channel ("chip select") to use. 
* All usable channels must have a sense of "connected/available" ("available" from here on).
  * USB-serial gets a signal when the host side has an active connection.
  * An SD card socket has a Card-Detect (CD) pin to know when an SD card is present.
  * An SPI socket has the !Interupt pin to detect the presence of an device on a select line.
* What about cases where there are multiple logical "control channels?" Example: SPI-connected "front panel" device while there's a USB serial connection open and the data channel is an sd card. We would like the front panel to be able to "pause" and "resume" as well as get status reports while the USB serial is is still getting status reports and can control via some UI there as well.
  * Possible solution: One channel is **Data**, and might also be a **Control** (UC_3 mode) but all other channels _that are `I`_ are _also_ **Control**. IOW, Control is a broadcast (status reports, etc) and accept-from-anywhere of commands, but will reject GCode from any channel not **Data**.
  * The first "available" `I` channel is the default Data.
  * It's possible that there is no current Data channel, such as when there's an `IC` front panel but no USB serial and no selected SD file to run from. 