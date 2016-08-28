This page is mostly a placeholder for now, but there are a few important pieces of information

We use Coolterm to connect to g2core for testing. 

## Drivers for g2core

### OSX
OSX you have everything you need.  The drivers will install automatically.

###Windows
Windows users need download an INF file in order to use g2core. Use this [G2 Windows Driver](https://raw.githubusercontent.com/synthetos/g2/edge/TinyGv2.inf) to download the inf file.  **Besure to "right click save as** when downloading. Note that the g2core USB registration makes it show up in the device list as `TinyG v2`.

Here is a video tutorial explaining how to install the g2core drivers on windows 7.<br>
[G2 Windows 7 Driver Installation Video](www.youtube.com/watch?v=UCF4FoVghsI)

## Connecting

* For OSX please use Coolterm version 1.4.3. There is a bug in the latest version and 1.4.3 is the latest we have tested that works well. (A bug has been opened with the author who has supported us extremely well throughout this project). Look on this page for this file: [CoolTermMac143](http://freeware.the-meiers.org/previous/)

* The usb will com up as 2 devices labeled something like:
  * usbmodem14311
  * usbmodem14313

These are the 2 endpoints for the dual endpoint USB connection. You can connect to either and it will work. The first port opened will become a control and data port. If you open the second port it will become a data port, and the first will become control-only. Send Gcode to the data port, and everything else to the control port. All responses, status reports and exception reports will be returned to the control port - no text will be sent to the data port. This way you can queue a lengthy gcode file to the data port, while still having access to controls.  