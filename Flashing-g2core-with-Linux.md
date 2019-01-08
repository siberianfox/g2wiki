This page assumes you have already built or downloaded ```gShield_flash.bin```.

Edit: 4/23/2015 Make is now building ```gShield.bin```, modify file name in commands as necessary

#### Step 1. Install BOSSA flash programming utility.

Most likely (on Debian or Ubuntu), you will need to type the following in the terminal:

```
sudo apt-get install bossa-cli
```
Alternatively, you could get this utility from [SourceForge](http://sourceforge.net/projects/b-o-s-s-a/).

#### Step 2. Connect Arduino Due via micro-usb cable.

Make sure, you use the Programming USB port.

#### Step 3. Activate bootloader:
```
stty -F /dev/ttyACM0 1200 hup
```
Note: This changes the default baud rate for port /dev/ttyACM0 to 1200, so if you leave the programming port and the native port both hooked up and the accidentally open /dev/ttyACM0 at a later time, you will erase your flash. So I do this instead:
```
stty -F /dev/ttyACM0 1200 hup
stty -F /dev/ttyACM0 9600
```
which I think will wind up erasing twice, but it leaves the default at 9600, so that if you accidentally open /dev/ttyACM0 then it won't wipe out your firmware.

#### Step 4. Flash the chip with `bossac`:
```
bossac  --port=ttyACM0  -U true -e -w -v -i -b -R ./bin/gShield/gShield_flash.bin
```
It will output something like:

```
Erase flash
Write 117704 bytes to flash
[==============================] 100% (460/460 pages)
Verify 117704 bytes of flash
[==============================] 100% (460/460 pages)
Verify successful
Set boot flash true
Device       : ATSAM3X8
Chip ID      : 285e0a60
Version      : v1.1 Dec 15 2010 19:25:04
Address      : 524288
Pages        : 2048
Page Size    : 256 bytes
Total Size   : 512KB
Planes       : 2
Lock Regions : 32
Locked       : none
Security     : false
Boot Flash   : true
CPU reset.
```

You're done. If everything went right, RX orange led should be slowly blinking. To connect to the board through serial, you will need to attach the cable to "Native USB" port.

If you're told that ```no device found on ttyACM0``` you may need to add yourself to the tty and dialout groups, try this:
```
sudo usermod -a -G tty $USER
sudo usermod -a -G dialout $USER
```
Then repeat steps 2 and 3.

I found that the following script allows me to leave both the programming port and the native port both hooked up, and flashes over the native port, which is much faster than flashing over the programming port.
I wrote a script called find_port.py which you can find here: https://github.com/dhylands/usb-ser-mon/blob/master/usb_ser_mon/find_port.py

```
#!/bin/bash
set -x
PORT=$(find_port.py --vid 2341)

# Activate the bootloader
stty -F ${PORT} 1200 hup
stty -F ${PORT} 9600
sleep 3

# Program
bossac -e -w -v -i -b -R ./bin/gShield/gShield.bin
```

It takes about 20 seconds after flashing before the native port becomes available for use. You can use: ```find_port.py --vid 1d50``` or ```find_port.py --vendor Synthetos``` to find the native port.

```find_port.py --list``` will show all connected USB-serial devices.