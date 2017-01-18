_This page is for uploading an already compiled G2 binary to a target board without a debugger from Windows. Please see [Getting Started with g2core Development](Getting-Started-with-g2core-Development) for information about other options._

###Step 1 - Get the TinyG2.bin file

**Option 1** - Compile your own using the instruction in [Compiling the Code](Getting-Started-with-g2core-Development#compiling-the-code)

**Option 2** - Get the tinyg2.bin binary firmware files from http://synthetos.github.io/g2/

In either case you'll need to note the path to the resulting binary file. For example, if it's called `g2core.bin` and it's in the Downloads directory of your home page, you would use `%HOMEPATH%\Downloads\g2core.bin`. We'll call this `PATH_TO_BINARY` in the commands below.

###Step 2 - Download the `arduino-flash-tools`

The [arduino-flash-tools](https://github.com/facchinm/arduino-flash-tools/tree/master) repo contains the `bossac` utility. You can download the [zip file here](https://github.com/facchinm/arduino-flash-tools/archive/master.zip).

Expand the zip file in a location where you know the path, and note that path.

###Step 3 - Program g2core using `bossac.exe`

Hold down the Windows Key and press r then type cmd.exe and hit enter:

* `cd ` *path to expanded zip file contents* <br>
* `cd tools_windows\bossac\bin` <br>

Run the `mode` command by itself and it should return a `COM`x.  Note your port number for x to use as `COMx` in the following commands. Remember from above that `PATH_TO_BINARY` is to be replaced with the path to the downloaded or compiled `.bin` file.

After that go ahead and run these commands, replacing `COMx` with your port, like this: `COM4`.
* `mode COMx BAUD=1200`
* `bossac.exe --port=COMx -U false -e -w -v -b PATH_TO_BINARY -R`
