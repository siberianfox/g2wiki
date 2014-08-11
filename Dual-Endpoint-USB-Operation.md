_NOTE: This page is currently a discussion about the specification for using two USB serial devices to drive G2 (a specifiction?). At some point it will be edited to become the specification._

This page describes using two USB serial devices with G2 - aka dual endpoint USB. 

##Theory Of Operation
The basic idea of dual endpoint USB operation is to have 2 independent channels for communication to the board. The first channel is a **control** channel through which machine commands are sent. The control channel is always active to process commands in near-real time. The second is a **data** channel through which the Gcode file is queued. 

This arrangement greatly simplifies flow control for the UI or host, as the control channel can be viewed as always being available to process incoming commands such as feedholds (stops), whereas the data channel can be  heavily buffered, and therefore always have the next Gcode block at the ready. 

##Requirements & Use Cases
The following use cases support most configurations.<br>
_Please note any other cases that are not covered._

* UC_1: Two USB serial ports - The USB has two virtual serial ports configured, control and data. This case also encompasses multiple simultaneous control channels.
 
* UC_2: One USB virtual serial port, one mass storage device - One virtual serial port is configured as the control channel. Data is made available from a mass storage device. Sub-cases include data channel being from (a) USB mass storage device, (b) SD card or similar off-board, serial accessed storage, (c) on-chip FLASH or EEPROM. Each case may assume read/write or read-only mass storage. This case also encompasses multiple simultaneous control channels.

* UC_3: One USB serial port - Both control and data are "piggybacked" on the same USB port ("device"). This mode is used to support v8 and earlier boards and cases where 2 endpoints are not available for some reason. Since only one port is used this case assumes only one control channel.

#Functional Specifications
##Channels & Devices
A **channel** is a logical communications channel such as ctrl or data. **Device** is a port or other data source that gets mapped to a channel. Right now we are mostly concerned with USB virtual serial port devices but devices may include:
  * USB virtual serial ports
  * USB mass storage devices
  * SD card and other mass storage devices
  * SPI channels and SPI connected devices
  * Other bulk storage or serial devices

The following are expected on the control and data channels.
###Control Channel
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
* Multiple control channels may be active at a time. Command input will be processed on a line-by-line basis (no character interleaves), first-come-first-serve, and round-robin. Responses are "broadcast" to all active control channels. This supports multiple control devices such as a desktop and a mobile pendant, or an SPI or USB connected front panel controller.

###Data Channel
* Data channel accepts all Gcode input:
  * Raw Gcode blocks
  * Blocks are read one line at a time
  * Lines terminated with LF, CR or both
  * All input is interpreted as Gcode (or whatever data protocol is in effect). All else is treated as an error.
  * _Note: JSON wrapped Gcode is not supported_ 
* Data channel returns:
  * No response is provided back on the data channel (no echo, acknowledgements, or errors)
* Only one data channel may be active at any given time.

###USB Communications and Channel Binding
G2 will implicitly bind channels using the available/connected state of the USB devices.

###Automatic Binding
G2 automatically binds the USB devices to the control and data channels. USB-serial subsystem is able to detect when software on the host side 'connects' to each channel independently. (This is done by a flag in the USB system that is set the the host connects and 'asserts RTS')

1. Initially neither USB device is assigned as control or data, and there is nothing connected to communicate with them anyway.

1. The first usb-serial device connected will be *both* control and data as long as it's the only device connected. This is to maintain compatibility with UC_3 and not require any additional setup steps for legacy UIs and hosts. 

1. If a second usb-serial channel is connected then the first channel becomes control-only and the second channel becomes data-only. This allows simple adoption of UC_1 mode by simply opening the two connections in a controlled order - the first connected will become command and the second connected will become data.

_Note: Multiple control ports and the use of other devices such as SD card files is supported using manual binding - more later._

###Automatic Unbinding
* If one of the two USB channels are closed then the other must become both data and control.
* If both channels are closed, then there is no data or control.

A rough version of dual-endpoint USB has been pushed to the alden branch for testing. There are some shortcuts taken, but it should behave more or less according the description in the above wiki page. In short, use it like this:

#Notes on the Current Implementation
_This section describes the current implementation, that may have some differences from the above specifiction. The current implementation is available as build 048.14 in the alden branch_

Steps are:

1. System is idle. Nothing is connected.
1. Connect to usbserial001. This will become a ctrl+data channel
1. Connect to usbserial003. This will become the data channel, and usbserial001 will become the control channel.
1. Send controls (JSON) to ctrl (001) and Gcode to data (003). The reader will prioritize controls over data and always read and process the control if one is available before attempting to read the next gcode block.
1. If data (003) disconnects the ctrl (001) reverts to being a ctrl+data channel. 003 can be re-connected as a data channel. If ctrl (001) disconnects then both channels disconnect.

That's it.

A few observations/questions:

* There is currently no distinction at the controller (reader) level between text lines returned by the control channel and lines returned by data (gcode blocks). The controller will happily receive both and execute both. I don't think this is a problem, but people should be aware of this.

* Any command received on either channel will send its responses to the control channel. Including received data. This may or may not be a good idea. Depends on use. Also, there is no output sent to the data channel. Feedback welcome.

* One possible enhancement would be to send a connect message back up the channel that was was just connected or changed. So the first connect would send a message indicating ctrl+data, the second would send a message over both ports, one for data, the other for control. SImilar messages would be send for other state transitions. This is more complexity, but if it's useful we should discuss. Part of this might also be the ability to query a channel for it's capabilities and state.

#Design and Implementation Notes
##Channels
* The two USB channels appear as physical devices:
  * `/dev/usb-serial0`
  * `/dev/usb-serial1`
* These are bound to the control and data logical devices:
  * `ctrl0`
  * `data0`

###Logical Channels
Logical channels are functions to which physical devices are attached (bound). These include:
* `ctrl0`, `ctrl1`, etc. Control channels. [See here for details](#control-channel)
* `data0`, `data1`, etc. Data channels [See here for details](#data-channel)

###Physical Devices
Physical devices are described by a fully qualified path. Most physical devices describe a single piece of hardware functionality, but some may be subset using filenames, slave select numbers, or other qualifiers. The following physical devices are known or supposed:
* `/dev/usb-serial0`, `/dev/usb-serial1`
* `/dev/uart0`, `/dev/uart1`, etc.
* `/dev/spi0`, `/dev/spi1`, etc. Describes entire SPI channel
* `/dev/spi0.0`, `/dev/spi0.1`, etc. Describes an endpoint (slave select) on an SPI channel
* `/dev/sd0` describes an SD card itself
* `/dev/sd0/file/filename` describes a file on an SD card
* `/dev/flash/addr/0x00000000` depending on the chip this may make sense
* `/dev/eeprom/addr/0x00000000` same
* `/dev/ram/addr/0x00000000` same

###Physical Device Read/Write/Interactive Capabilities
* `R/W` devices are physical devices that are read/write
* `R` devices are read-only (plus flow control)
* `I` devices are a subset of `RW` devices that are interactive and capable of sending commands (JSON, etc), receiving status reports, and sending GCode
* `IC` devices are a subset of interactive devices that cannot transmit GCode such as a front-panel device.
* Examples:
  * A USB serial device is default `I`. It will become `RW` when set as a data channel. Writing to a data channel has the effect of storing or piping the contents, depending on the device. _(Is this true? - alden)_
  * An SD card is either `R` or `RW`, but never `I`. (Note: You still talk to an SD card to get data, even when the card is `R`.)
* An SPI device may be `R`, `RW`, `I`, or `IC`. There needs to be a mechanism to determine which. 

###Physical Device Behaviors
* When an non-`I` device, such as an SD card, is the data channel it's expected to automatically keep the gcode buffer full until there is no more (EOF, etc).
* When selecting a data channel the rest of the path beyond that necessary to designate the device itself, if any, will be used to determine a subset of the channel. Example:
  * "/sd/file/path" will pass "/file/path" to the sd channel. It might not *accept* the Data channel designation (file not found, card missing, etc).
  * "/spi0/cs1" will pass "/cs1" to the SPI device for determining which sub-channel ("chip select") to use. 
* All usable channels must have a sense of "connected/available" ("available" from here on).
  * USB-serial gets a signal when the host side has an active connection.
  * An SD card socket has a Card-Detect (CD) pin to know when an SD card is present.
  * An SPI socket has the !Interrupt pin to detect the presence of an device on a select line.

_Note: Need a definition of the state model. To include connected/available, active, etc._

_Note 2: Motate will hide the chip-select semantics, so consider their use here as merely example. In the end, it will just appears as a series of SPI<b>n</b> channels._

###Multiple Control Channels
What about cases where there are multiple logical "control channels?" Example: SPI-connected "front panel" device while there's a USB serial connection and the data channel is an sd card. We would like the front panel to be able to "pause" and "resume" as well as get status reports while the USB serial is is still getting status reports and can control via some UI there as well.
  * Possible solution: One channel is **Data**, and might also be a **Control** (UC_3 mode) but all other channels _that are `I`_ are _also_ **Control**. IOW, Control is a broadcast (status reports, etc) and accept-from-anywhere of commands, but will reject GCode from any channel not **Data**.
  * The first "available" `I` channel is the default Data.
  * It's possible that there is no current Data channel, such as when there's an `IC` front panel but no USB serial and no selected SD file to run from. 

#Filesystem Commands

All filesystem commands will be in the `{fs:{...}}` namespace.

* As for all commands in the G2 system, queries/commands and responses are loosely coupled.
* When a query is received, one or more responses are sent out.
* The response format will be dictated by the request. That format will either implicitly indicate that there will be no more responses (such as when only one was expected), or will explicitly indicate which response is the last.
* There _might_ be other information - such as status reports - transmitted by G2 between the request and the response or between responses.

#SD Card Operation
## SD Card Use Cases

* UC_SD1: Return card parameters
  * Query the device, for **device** info:
  ```javascript
  // Query
  {fs:{info:"/dev/sd"}}
  // Possible responses:
    // Card present
    {fs:{info:{device:"/dev/sd", mount:"/sd", available:true}}}
    // Card missing/ejected
    {fs:{info:{device:"/dev/sd", available:false}}}
  ```
    * Note 1: The string used in the info request must be the string used in the response, even if several possible paths would lead to the same physical device.
  * Query the mount for **card** info:
  ```javascript
  // Query:
  {fs:{info:"/sd"}}
  // Possible responses:
    // Card present
    {fs:{info:{mount:"/sd", available:true, volume:"UNAMED_CARD", size:4194304, read:true, write:false}}}
    // Card missing/ejected
    {fs:{info:{mount:"/sd", available:false}}}
  ```

* UC_SD2: Retrieve directory listing
  ```javascript
  // Query (list "/sd", only list the first five items)
  {fs:{ls:"/sd", first:null, count:5}}
  // Possible responses:
    // Files may or may not be sorted.
    // If unsorted, the order may change on any change to the filesystem.
    // Reply must use the same string as the request, even if several
    //  paths may lead to the same directory.
    {fs:{ls:"/sd", item:"test.gc", file:true}}
    {fs:{ls:"/sd", item:"braid.gcode", file:true}}
    {fs:{ls:"/sd", item:"image.jpg", file:true}}
    {fs:{ls:"/sd", item:"test.gc", file:true}}
    {fs:{ls:"/sd", item:"old_stuff", directory:true}}
  
    // A null item indicates that there will be no more responses for
    //  this listing request. 'more' being true means there are more
    //  items to list. 'next' is the token you would provide as "first"
    //  to get the next group of items. It should not be considered
    //  an index into an array.
    {fs:{ls:"/sd", item:null, more:true, next:1155}}

  // Query (list "/sd", only list the next five items)
  {fs:{ls:"/sd", first:1155, count:5}}
    // The next five (or less) {fs:{ls:...}} responses
    {fs:{ls:"/sd", item:null, more:false}}
  ```

* UC_SD3: Read a data file from SD card. A file on the SD card is used as a data source (data channel) to deliver a Gcode program. The following behaviors and limitations apply:
  * When the SD file is selected the code will start executing automatically (Do we want this behavior or do we want an explicit cycle start?)
    * Is there a reason not to? It could be simulated with an M0 as the first item in the file.
  * When the program is complete the data channel reverts to the data channel that was in effect at the time the file was selected. 
  * A program completes when either an M2 or M30 is encountered (Gcode Program End), or an end-of-file is encountered.
  * Embedded Stops (M0, M1) will cause the program to halt at that point. Program execution resumes with a Resume (cycle start) being received on any control channel.

* UC_SD4: Write a data file to an SD card

* UC_SD5: Stream a data file to an SD card

* UC_SD6: Read a control file from an SD card
  * What would a control file be?
* UC_SD7: Write a control file to an SD card


## Notes and Open Questions
* Nested devices: 
  * How would we describe an sd card that is on a fin at `spi0.1`?
  * How about an individual websocket on a WiFi fin?
* "Teletype" Channels: `tty0`, `tty1`. Do we need these?
* Good FAT16 reference http://elm-chan.org/fsw/ff/00index_e.html