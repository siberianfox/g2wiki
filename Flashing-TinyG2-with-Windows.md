Flashing TinyG2 to the Arduino Due w/ Windows
=============================

###Step 1 - Get the TinyG2.bin file
Get the tinyg2.bin binary firmware files from the link below.
<br>
http://synthetos.github.io/g2/<br>
Get the **USB Test for Due - default settings** bin file.

###Step 2 - Install the Arduino Due environment

_If you already have the Arduino Due environment installed you can skip this step_

Download the Arduino 1.5 BETA (not the 1.0.5 release at the top of the page) from http://arduino.cc/en/Main/Software for Windows.

Get the "installer" and run installer executable.

###Step 3 - Program TinyG2 onto the Due

Hold down the Windows Key and press r then type cmd.exe and hit enter:

If you are on Windows x64 then run this command.
* `cd %ProgramFiles% (x86)\Arduino\hardware\tools`

If you are on Windows x32 then run this command.
* `cd %ProgramFiles%\Arduino\hardware\tools`
* `mode COM6 BAUD=2400'
* `bossac.exe --port=COM6 -e -w -v -b %HOMEPATH%\Downloads\http://synthetos.github.io/g2/binaries/TinyG2_Due_rob_usbtest.bin -R`

Note that COM4 is the port that my ArduinoDUE showed up as.  You can run the `mode` command by itself and it should return a COMx.  Just use your port number.