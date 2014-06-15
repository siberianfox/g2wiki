This page assumes you have already built or downloaded ```gShield_flash.bin```.

####Step 1. Install BOSSA flash programming utility.

Most likely (on Debian or Ubuntu), you will need to type the following in the terminal:

```
sudo apt-get install bossa-cli
```
Alternatively, you could get this utility from [SourceForge](http://sourceforge.net/projects/b-o-s-s-a/).

####Step 2. Connect Arduino Due via micro-usb cable.

Make sure, you use the Programming USB port.

####Step 3. Activate bootloader:
```
stty -F /dev/ttyACM0 1200
```
####Step 4. Flash the chip with `bossac`:
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