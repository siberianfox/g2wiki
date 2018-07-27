To connect to g2core you must have the correct USB drivers on your host computer. We use the [g2core-api](https://github.com/synthetos/node-g2core-api) NodeJS module or [Coolterm v1.4.3](http://freeware.the-meiers.org/previous/) for testing g2core. [Chilipeppr](https://github.com/synthetos/tinyg/wiki/Chilipeppr) is a good option for a CNC controller. Other options are [Universal G-code Sender](https://github.com/synthetos/g2/wiki/Universal-Gcode-Sender), [Goko](http://goko.fr/) or [cncjs](https://cnc.js.org/)

See the [g2core Communications](g2core-Communications) page if you need details of the communications protocol. If you are using one of the above communications tools you do not need the page.

## Drivers for g2core

### OSX
OSX you have everything you need.  The drivers will install automatically.

### Windows
Windows users need download an INF file in order to use g2core. Use this [G2 Windows Driver](https://raw.githubusercontent.com/synthetos/g2/edge/Resources/TinyGv2.inf) to download the inf file.  **Besure to "right click save as** when downloading. Note that the g2core USB registration makes it show up in the device list as `TinyG v2`.

Here is a video tutorial explaining how to install the g2core drivers on windows 7.<br>
www.youtube.com/watch?v=UCF4FoVghsI

Windows 10 does not need this procedure and should work out-of-the-box.


## Connecting

### Option 1: g2core-api
Use the [g2core-api](https://github.com/synthetos/node-g2core-api) NodeJS module to connect to g2core, run command lines, and send files. The Readme at the above link explains how to install and use. 

### Option 2: Coolterm 1.4.3
For OSX you can also use Coolterm version 1.4.3. There is a bug in the latest version and 1.4.3 is the latest we have tested that works well. (A bug has been opened with the author who has supported us extremely well throughout this project). Look on this page for this file: [CoolTermMac143](http://freeware.the-meiers.org/previous/)

* In Coolterm the usb will show up as 2 devices labeled something like:
   * usbmodem14311
   * usbmodem14313

These are the 2 endpoints for the dual endpoint USB connection. You can connect to either and it will work. For most purposes that's all you need.

If you want to run dual port mode, the first port opened will become a control and data port. If you open the second port it will become a data port, and the first will become control-only. Send Gcode to the data port, and everything else to the control port. All responses, status reports and exception reports will be returned to the control port - no text will be sent to the data port. This way you can queue a lengthy gcode file to the data port, while still having access to controls.
