_NOTE: This page is currently a discussion about the specification for using 2 USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first channel is a **control** channel through which machine commands are sent. The control channel is always open to process commands in near-real time. The second is a **data** channel through which the Gcode file is queued. 

This arrangement greatly simplifies flow control for the UI or host, as the control channel can be viewed as always being available to process incoming commands such as feedholds (stops), whereas the data channel can be  heavily buffered, and therefore always have the next Gcode block at the ready. 

##Requirements & Use Cases
The following use cases support most configurations. _Please note any other cases that are not covered._

* UC_1: Two USB serial ports - The USB has two virtual serial ports configured, control and data. This case also encompasses multiple simultaneous control channels.
 
* UC_2: One USB virtual serial port, one mass storage device - One virtual serial port is configured for the control channel. Data is made available from a mass storage device. Sub-cases include data channel being from (a) USB mass storage device, (b) SD card or similar off-board, serial accessed storage, (c) on-chip FLASH or EEPROM. Each case may assume read/write or read-only mass storage. This case also encompasses multiple simultaneous control channels.

* UC_3: One USB serial port - Both control and data are "piggybacked" on the same USB port. This mode is used to support v8 and earlier boards and cases where 2 endpoints are not available for some reason. In this case only one port is supported.

##Functional Specifications

###Functions
A **channel** is any communications channel to the board. We are mostly concerned with USB virtual serial ports at this point but channels may include:
  * USB virtual serial ports
  * USB mass storage devices
  * SD card and other mass storage devices
  * SPI channels and SPI connected devices
  * Other devices

The following are expected on the control and data channels.
* **Control Channel**
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
  * Multiple control channels may be open at a time. Command input will be processed on a line-by-line basis (no character interleaves), first-come-first-serve, and round-robin. Responses are "broadcast" to all open control channels. This supports multiple control devices such as a desktop and a mobile pendant, or an SPI or USB connected front panel controller.

* **Data Channel**
  * Data channel accepts all Gcode input:
    * Raw Gcode blocks
    * Blocks are read one line at a time
    * Lines terminated with LF, CR or both
    * All input is interpreted as Gcode (or whatever data protocol is in effect). All else is treated as an error.
    * _Note: JSON wrapped Gcode is not supported_ 
  * Data channel returns:
    * No response is provided back on the data channel (no echo, acknowledgements, or errors)
  * Only one data channel may be open at any given time.


###USB Communications and Channel Binding
G2 will implicitly bind channels using the available/connected state of the USB channels.

**Background**<br>
* The two USB channels appear as physical devices:
  * `/dev/usb-serial0`
  * `/dev/usb-serial1`
* These are bound to the control and data logical devices:
  * `ctrl0`
  * `data0`

**Automatic Binding**<br>
G2 automatically binds the USB physical devices to the logical devices. USB-serial subsystem is able to detect when software on the host side 'connects' to each channel independently. (This is done by a flag in the USB system that is set the the host connects and 'asserts RTS')

1. Initially neither USB channel is assigned as control or data, and there is nothing connected to communicate with them anyway.

1. The first usb-serial channel opened (connected to) will be *both* control and data as long as it's the only channel open. This is to maintain compatibility with UC_3 and not require any additional setup steps for legacy UIs and hosts. 

1. If a second usb-serial channel is opened then the first channel becomes control-only and the second channel becomes data-only. This allows simple adoption of UC_1 mode by simply opening the two connections in a controlled order - the first opened will become command and the second opened will become data.

_Note: Multiple control ports and the use of other devices such as SD card files is supported using manual binding - more later._

**Automatic Unbinding**<br>
* If one of the two USB channels are closed then the other must become both data and control.
* If both channels are closed, then there is no data or control.

##Design and Implementation Notes

**Physical Devices**<br>
Physical devices are described by a fully qualified path. Most physical devices describe a single piece of hardware functionality, but some may be subset using filenames, slave select numbers, or other qualifiers. The following physical devices are known or supposed:
* `/dev/usb-serial0`, `/dev/usb-serial1`
* `/dev/uart0`, `/dev/uart1`, etc.
* `/dev/spi0`, `/dev/spi1`, etc. Describes entire SPI channel
* `/dev/spi0.0`, `/dev/spi0.1`, etc. Describes an endpoint (slave select) on an SPI channel
* `/dev/sd0` describes an SD card itself
* `/dev/sd0/filename` describes a file on an SD card

**Logical Channels**<br>
Logical channels are functions to which physical devices are attached (bound). These include:
* `ctrl0`, `ctrl1`, etc. Control channels (multiple may be active at a time)
* `data0`, `data1`, etc. Data channels (only one may be active at a time)

**Physical Device Read/Write/Interactive Capabilities**<br>
* `R/W` devices are physical devices that are read/write
* `R` devices are read-only (plus flow control)
* `I` devices are a subset of `RW` devices that are interactive and capable of sending commands (JSON, etc), receiving status reports, and sending GCode
* `IC` devices are a subset of interactive devices that cannot transmit GCode such as a front-panel device.
* Examples:
  * A USB serial device is default `I`. It will become `RW` when set as a data channel. Writing to a data channel has the effect of storing or piping the contents, depending on the device. _(Is this true? - alden)_
  * An SD card is either `R` or `RW`, but never `I`. (Note: You still talk to an SD card to get data, even when the card is `R`.)
* An SPI device may be `R`, `RW`, `I`, or `IC`. There needs to be a mechanism to determine which. 

**Physical Device Behaviors**<br>
* When an non-`I` device, such as an SD card, is the data channel it's expected to automatically keep the gcode buffer full until there is no more (EOF, etc).
* When selecting a data channel the rest of the path beyond that necessary to designate the device itself, if any, will be used to determine a subset of the channel. Example:
  * "/sd/file/path" will pass "/file/path" to the sd channel. It might not *accept* the Data channel designation (file not found, card missing, etc).
  * "/spi0/cs1" will pass "/cs1" to the SPI device for determining which sub-channel ("chip select") to use. 
* All usable channels must have a sense of "connected/available" ("available" from here on).
  * USB-serial gets a signal when the host side has an active connection.
  * An SD card socket has a Card-Detect (CD) pin to know when an SD card is present.
  * An SPI socket has the !Interrupt pin to detect the presence of an device on a select line.

_Note: Need a definition of the state model. To include connected/available, active, etc._

**Logical Channels**
* The control channel is where G2 reads commands. See here a compound entity that is made of one or more 

**Physical Device Behaviors**<br>

* What about cases where there are multiple logical "control channels?" Example: SPI-connected "front panel" device while there's a USB serial connection open and the data channel is an sd card. We would like the front panel to be able to "pause" and "resume" as well as get status reports while the USB serial is is still getting status reports and can control via some UI there as well.
  * Possible solution: One channel is **Data**, and might also be a **Control** (UC_3 mode) but all other channels _that are `I`_ are _also_ **Control**. IOW, Control is a broadcast (status reports, etc) and accept-from-anywhere of commands, but will reject GCode from any channel not **Data**.
  * The first "available" `I` channel is the default Data.
  * It's possible that there is no current Data channel, such as when there's an `IC` front panel but no USB serial and no selected SD file to run from. 

## Manual Binding
The automatic binding operations hide the details of the device internals. Manual device manipulation is available for more complex cases. The model is to map physical devices to logical devices (functions).

Logical devices are described by a fully qualified path. The following logical devices are known or supposed:
  * `/dev/usb-serial0`, `/dev/usb-serial1`
  * `/dev/uart0`, `/dev/uart1`,etc.
  * `/dev/spi0`, `/dev/spi1`, etc. Describes entire SPI channel
  * `/dev/spi0.0`, `/dev/spi0.1`, etc. Describes an endpoint (slave select) on an SPI channel
  * `/dev/sd0` describes an SD card
  * `/dev/sd0/filename` describes a file on an SD card


* These are mapped to the control and data logical devices:
  * `ctrl0`
  * `data0`
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

### Multiple Device Channels
