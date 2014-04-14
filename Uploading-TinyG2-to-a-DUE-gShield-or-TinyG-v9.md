_Work in progress._

# Locating or creating the binary to upload

You must either download or build the binary to be uploaded to the board. Make sure you get the correct binary for the board you are uploading to. For a DUE with a gShield (a.k.a. GRBL Shield), it will be labelled as "gShield." For any version of TinyG v9 (prototypes, at this point), it will be labelled with the prototype revision, such as  "G2v9g."

# Activating the ATSAME3X8* Loader

The ATSAM3X8C and ATSAM3X8E processors we're using have a built-in loader that in the ROM. In order to activate this loader, you need to tell the board to reboot into it.

There are two ways to do this:

1. On boards that are equipped with an Erase button (DUE) or Erase jumper (v9g and above), you can either hold down the erase button (or set the erase jumper) while hitting the reset button. _**Warning:** The board will continue to boot into the loader until you program it. It will, effectively, be "erased."_ Once it's booted into the loader, release the button or remove the jumper. The board is now in the loader, and ready to be programmed. _Remember to remove the jumper, or the board will be erased on the next reset!_

1. For boards that already have an Arduino firmware installed (the DUE) or have a version of TinyG2 installed that's newer than version 27.0, then it will reset into the loader _until the next reset_ when you open and close a USB-serial connection to the board at 1200 baud.

On **OSX**, connect only _one_ USB-serial device to your machine then, in `Terminal.app`:

```bash
ls /dev/tty.usbmodem*
# There should be only one item listed if you have one usb-serial device connected.
stty -f /dev/tty.usbmodem* 1200
```

On **Linux**, the instructions should be the same or similar to OSX. (TODO: Test this.)

On **Windows**, connect only _one_ USB-serial device to your machine then, look up the com port number in the device manager (will either say "Arduino Due" or "TinyG v2"). Then, assuming _COM6_ (change it accordingly), in `cmd.exe`:

```batch
mode COM6 BAUD=1200
```

# Programming the board with BOSSAc

In order to program the board now that it's reset into the loader, you will need to download the [Arduino IDE](http://arduino.cc/en/Main/Software#toc3). Note that this _must_ be the version that supports the Due, which is not currently the release version.


TODO: How to convert an ELF to a BIN.

TODO: Upload the BIN with OS X. Basis: `$arduinoAppDir/Arduino.app/Contents/Resources/Java/hardware/tools/bossac -e -w -v -b "${file/.elf/.bin}"`

TODO: Upload the BIN with Windows. Basis: `bossac.exe --port=COM6 -e -w -v -b %HOMEPATH%\file_path.bin -R`
