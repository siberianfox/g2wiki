_This page is for uploading an already compiled G2 binary to a target board without a debugger from Windows. Please see [Getting Started with G2](Getting-Started-with-G2) for information about other options._

###Step 1 - Get the TinyG2.bin file

**Option 1** - Compile your own using the instruction in [[Compiling G2 on Windows (Atmel Studio 6.2)]]

**Option 2** - Get the tinyg2.bin binary firmware files from http://synthetos.github.io/g2/

###Step 2 - Install the Arduino Due environment

_If you already have the Arduino Due environment installed you can skip this step_

Download the Arduino 1.6.x from http://arduino.cc/en/Main/Software for Windows.

Get the "installer" and run installer executable.

###Step 3a - Program TinyG2 onto the Due Using Arduino Tools

Hold down the Windows Key and press r then type cmd.exe and hit enter:

If you are on Windows x64 then run this command.
* `cd %ProgramFiles% (x86)\Arduino\hardware\tools`

If you are on Windows x32 then run this command.
* `cd %ProgramFiles%\Arduino\hardware\tools`<br>
After that go ahead and run these commands.  You should be good to go.
* `mode COM6 BAUD=1200`
* `bossac.exe --port=COM6 -e -w -v -b %HOMEPATH%\Downloads\TinyG2_Due_rob_usbtest.bin -R`

Note that COM4 is the port that my ArduinoDUE showed up as.  You can run the `mode` command by itself and it should return a COMx.  Just use your port number.

###Step 3b - Program TinyG2 onto the Due Using Atmel-ICE
This requires Atmel Studio6.2 (or later) and an Atmel-ICE programmer. I've omitted a lot of steps, but there are some things to watch for

* Make sure the ribbon cable is plugged into the SAM port, not the AVR port
* Select SWD, not JTAG
* Program the fuses to boot from FLASH, not ROM
