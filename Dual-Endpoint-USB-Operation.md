_NOTE: This page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first channel is a **control** channel through which machine commands are sent. The control channel is always open to process commands in near-real time. The second is a **data** channel through which the Gcode file is queued. 

This arrangement greatly simplifies flow control for the UI or host, as the control channel can be viewed as always being available to process incoming commands such as feedholds (stops), whereas the data channel can be  heavily buffered, and therefore always have the next Gcode block at the ready. 

##Requirements & Use Cases

* UC_1: Two USB serial ports - The USB has two virtual serial ports configured, control and data.
 
* UC_2: One USB Serial Port, One mass storage device - One virtual serial port is configured for the control channel. Data is made available from a mass storage device. Sub-cases include data channel being from (a) USB mass storage device, (b) SD card or similar off-board, serial accessed storage, (c) on-chip FLASH or EEPROM. Each case may assume read/write or read-only mass storage.

* UC_3: One USB serial port - Both control and data are "piggybacked" on the same USB port. This mode is used to support v8 and earlier boards and cases where 2 endpoints are not available for some reason.

##Functional Specifications

###Functions
* **Channels** are any communications channel. We are mostly concerned with USB virtual serial ports at this point but channels may include:
  * USB virtual serial ports
  * USB mass storage devices
  * SD card and other mass storage devices
  * SPI connected devices
  * Other devices
The following are expected on the control and data channels.
* Control Channel
  * Control channel accepts all non-gcode commands and controls, including:
    * Configuration commands (JSON and text mode)
    * Single-character controls: !, %, ~
    * multi-character controls such as feed rate overrides and other in-cycle commands
  * Control channel returns all responses from both channels, including
    * JSON response objects
    * JSON status reports
    * JSON exception reports
    * text-mode responses of all kinds
    * Gcode echo (if enabled)
    * Gcode comment messages
  * Multiple control channels may be opened at a time. Command input will be processed line-by-line (no character interleaves), first-come-first-serve, and round-robin. This supports multiple control devices such as a desktop and a mobile pendant, or an SPI connected front panel controller.

* Data Channel
  * Data channel accepts all Gcode input:
    * Raw Gcode blocks
    * Blocks are read one line at a time
    * Lines terminated with LF, CR or both
    * All input is interpreted as Gcode (or whatever data protocol is in effect). All else is treated as an error.
  * Data channel returns:
    * No response is provided back on the data channel (no echo, acknowledgements, or errors)

###USB Communications and Channel Binding
We can implicitly bind the channels using the available/connected state of the USB channels.

**Background**<br>
* The two USB channels appear as physical devices:
  * `/dev/usb-serial0`
  * `/dev/usb-serial1`
* The USB-serial subsystem is able to detect when software on the host side "connects" to each channel independently. (This is done by a flag in the USB system that is set the the host connects and 'asserts RTS')

**Binding**

1. Initially neither USB channel is assigned as control or data, and there is nothing connected to communicate with them anyway.

1. The first usb-serial channel opened (connected to) will be *both* control and data as long as it's the only channel open. This is to maintain compatibility with UC_3 and not require any additional setup steps for legacy UIs and hosts. 

1. If a second channel is opened, then the first channel becomes control-only, and the second channel becomes data-only. This allows simple adoption of UC_1 mode by simply opening the two connections in a controlled order, and the first opened will become command and the second opened will become data.

Other devices such as SD card files can also be bound to the data channel - more later.

**Unbinding**<br>
* If one of the two USB channels are closed then the other must become both data and control.
* If both channels are closed, then there is no data or control.

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


## Device mapping

###Things we need to describe
* What's a logical device and what's a physical device?
  * How do we keep them distinct?
* Serial devices: `usb-serial0`, `usb-serial1`, `uart0`, `spi0.1`
 * Subdevices: `spi0.1` would be sub-channel `1` of peripheral `spi0`?
 * Nested devices: 
    * How would we describe an sd card that is on a fin at `spi0.1`?
    * How about an individual websocket on a WiFi fin?
* "Teletype" Channels: `tty0`, `tty1`
* File storage devices: `sd`, `flash`, `ram`
  * Files, stored on those devices
