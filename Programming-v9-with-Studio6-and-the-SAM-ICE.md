_These are some really sketchy notes cribbed from a chat we were having._
We are assuming you are using VMware Fusion 6 under OSX. If not, some of these instructions won't be relevant and can be ignored.

### Setup

1. Bring up Atmel Studio 6.1. If the project complains that you don't have the complete project or some parts are not found don't worry - you don't need it if all you are doing is programming (and not compiling).

### Programming

1. Select `Tools`/`Device Programming`

1. First, make sure the SAM-ICE is plugged in via USB. 

1. Next, make sure the SAM-ICE is connected to the virtual machine. In the Virtual Machine/USB & Bluetooth menu, select `Connect SEGGER J-Link`

_NOTE: If anywhere in these next steps you get a dialog asking to upgrade the SAM-ICE firmware, do it._

Select `SAM-ICE` for the tool
Select `ATSAM3X8C` for the device _(Note: Use ATSAM3X8E if you are programming the Due)_
Select `SWD` for the Interface

Hit `Apply`. You may get a `Device Signature` coming back - about 8 hex digits. The `Target Voltage` should also say 3.3 volts, plus or minus. If not, hit `Read`

Click the `Memories` line in the left nav. Leave all settings at their defaults (Erase Chip, boxes checked)

Navigate to the TinyG2.elf file in the file select box

Hit `Program`

Close the programming dialog box and connect the board USB to the mac (it can be plugged in when you are programming, also). Make sure the USB port is recognized by the mac, not the VM (VMware or Parallels).

If you turn on a terminal window in Coolterm you should see something like usbmodem001

Connect at 115,200 - or anything other than 1200. USB don't care. USB just takes what it needs.

Hit ? in the interface and see if you get a status screen back
