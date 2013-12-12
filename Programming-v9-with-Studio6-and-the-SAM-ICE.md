Bring up Studio 6

Select `Tools`/`Device Programming`

Select `SAM-ICE` for the tool
Select `ATSAM3X8C` for the device
Select `SWD` for the Interface

Hit `Apply`. You should get a `Device Signature` coming back. The `Target Voltage` should also say about 3.2 or 3.3 volts

Click the Memories line in the left nav. Leave all settings at their defaults (Erase Chip, boxes checked)

Navigate to the elf file in the file select box

Hit `Program`

Close the dialog box and connect the board to the mac. Make sure the USB port is recognized by the mac, not Parallels

If you turn on a terminal window in Coolterm you should see something like usbmodem001

Connect at 115,200

Hit ? in the interface and see if you get a status screen back
