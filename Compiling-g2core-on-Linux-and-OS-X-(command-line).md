# What's Needed

* Linux (Ubuntu-based)
  ```bash
  sudo apt-get install git-core make
  ```

  If you're running a 64-bit version of Ubuntu, you'll also need to install some 32-bit pieces:

  ```bash
  sudo apt-get install libstdc++6:i386
  ```

* OS X - If you don't already have Xcode command line tools installed (it doesn't hurt to run it again):
  ```bash
  xcode-select --install
  ```

# Cloning the repo

  * See [Getting Started with G2core → Cloning the Repo](https://github.com/synthetos/g2/wiki/Getting-Started-with-g2core-Development#cloning-the-repo) if you haven't already.

# Build the sources:

For Arduino Due:
```
cd g2/g2core
make PLATFORM=DUE BOARD=gShield
```

For TinyG V9 board:
```
cd g2/g2core
make CONFIG=TestV9
```

* See [Adding and Revising Boards](Adding-and-Revising-Boards) for explanation of the available `make` options.

# Flashing

- See [[Flashing g2core with Linux]].

# JTag Debugging

We recommend using the [Segger J-Link](https://www.segger.com/products/debug-probes/j-link/) for debugging.  It can also be used for flashing (which may be faster than other flashing methods).

In addition to the debugger, you may also need a [SWD Cable](https://www.adafruit.com/product/1675) and [JTag to SWD Adapter](https://www.adafruit.com/product/2094) to connect to the Arduino Due.  Some versions of the J-Link already have a SWD port and come with a cable.

When connecting everything, ensure that all cables are plugged in with the correct polarity.  The SWD cable is not keyed, so markings on the boards should be used to align it.  Typically, there is a dot or `1` near one corner of the SWD pin header (look for a 5x2 pin header with 0.5mm pitch) that should be aligned with the striped side of the cable.

## Linux/Ubuntu Setup and Usage

1. Download and install the [J-Link Software and Documentation Pack](https://www.segger.com/downloads/jlink/#J-LinkSoftwareAndDocumentationPack).  If using Ubuntu, you probably want the 64-bit .deb.
2. Ensure your user account has access to usb/serial devices: `sudo usermod -a -G dialout USERNAME`
3. Install the multi-architecture version of gdb: `sudo apt-get install gdb-multiarch`
4. Start the JLink GDB server.  This command works for Arduino Due; it may need to be modified for other boards: `JLinkGDBServer -if SWD -device AT91SAM3X8E`
5. Start gdb: `gdb-multiarch ./bin/gShield/g2core.elf`
6. Inside gdb, connect to the JLink GDB server: `target extended-remote :2331`
7. Type `load` to flash the current binary to the device.
8. Type `monitor reset` to reset the device.
9. Type `continue` to run, and use gdb as normal.

You may need to re-plug the device's USB after flashing/resetting if the host is not properly detecting it immediately.
