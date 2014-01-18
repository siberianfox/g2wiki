_These are some really sketchy notes cribbed from a chat we were having._
We are assuming you are using VMware Fusion 6 under OSX. If not, some of these instructions won't be relevant and can be ignored.

### Setup

1. Bring up Atmel Studio 6.1. If the project complains that you don't have the complete project or some parts are not found don't worry - you don't need it if all you are doing is programming (and not compiling).

### Programming
_NOTE: If anywhere in these next steps you get a dialog asking to upgrade the SAM-ICE firmware, do it._

1. Select `Tools`/`Device Programming`

1. First, make sure the SAM-ICE is plugged in via USB. 

1. Next, make sure the SAM-ICE is connected to the virtual machine. In the Virtual Machine/USB & Bluetooth menu, select `Connect SEGGER J-Link`

1. Select `SAM-ICE` for the tool
1. Select `ATSAM3X8C` for the device for the v9, use ATSAM3X8E if you are programming the Due
1. Select `SWD` for the Interface

1. Hit `Apply`. You may get a `Device Signature` coming back - about 8 hex digits. The `Target Voltage` should also say 3.3 volts, plus or minus. If not, hit `Read`

1. Click the `Memories` line in the left nav. Leave all settings at their defaults (Erase Chip, boxes checked)

1. Use the file select box `...` to navigate to the TinyG2.elf file

1. Hit `Program` You should see a series of progress bars in the device window and also in the popup window driven by the SAM-ICE

1. Close the programming dialog box

### Connect the board to the mac

1. If you do not already have Coolterm get it here and install it http://freeware.the-meiers.org/

1. Connect the board USB to the mac (it can be plugged in when you are programming, also). Make sure the USB port is recognized by the mac, not the VM (VMware or Parallels). When you connect the USB you might see a VMware dialog asking where to connect the TinyG. Choose the mac.

1. You can verify that the USB was recognized by the mac by looking for it in the system report. Look here: <Apple> / About This Mac / More Info... / System Report ... / USB. You should see TinyG v2 in the device list. You can refresh this list by hitting `<command> r`. Make sure you see the TinyG v2 device.

1. Select the `Options' menu in Coolterm. '`Re-Scan Serial Ports`. You should see usbmodem001. If not, go back to the system report, and also make sure the TinyG is not in the VMware USB devices. If so, disconnect it.

Connect at 115,200 - or anything other than 1200. USB don't care. USB just takes what it needs.

Hit ? in the interface and see if you get a status screen back
