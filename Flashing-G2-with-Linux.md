This page assumes you have already built or downloaded TinyG2.hex.

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
bossac  --port=ttyACM0  -U  false -e -w -v -b -R TinyG2.hex
```
It will output something like:

```
Erase flash
Write 322289 bytes to flash
[==============================] 100% (1259/1259 pages)
Verify 322289 bytes of flash
[==============================] 100% (1259/1259 pages)
Verify successful
Set boot flash true
CPU reset.
```

You're done.