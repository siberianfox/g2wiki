_These are some really sketchy notes cribbed from a chat we were having._

1. Bring up Atmel Studio 6.1. If the project complains that you don;t have the complete project don;t worry - you don't need it if all you are doing is programming (and not compiling).

Select `Tools`/`Device Programming`

Select `SAM-ICE` for the tool
Select `ATSAM3X8C` for the device
Select `SWD` for the Interface

Hit `Apply`. You should get a `Device Signature` coming back - about 8 hex digits. The `Target Voltage` should also say 3.3 volts, plus or minus.

Click the `Memories` line in the left nav. Leave all settings at their defaults (Erase Chip, boxes checked)

Navigate to the TinyG2.elf file in the file select box

Hit `Program`

Close the programming dialog box and connect the board USB to the mac (it can be plugged in when you are programming, also). Make sure the USB port is recognized by the mac, not the VM (VMware or Parallels).

If you turn on a terminal window in Coolterm you should see something like usbmodem001

Connect at 115,200 - or anything other than 1200. USB don't care. USB just takes what it needs.

Hit ? in the interface and see if you get a status screen back
