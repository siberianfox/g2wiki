##Compiling OpenOCD:

You will need:
* Raspberry Pi
* 6 [Female-to-Female jumper wires](http://www.adafruit.com/products/266)
* Two [Small pin clips](http://www.adafruit.com/products/401) (IC Hooks)

*BNote:* I am using [Occidentalis v0.2](http://learn.adafruit.com/adafruit-raspberry-pi-educational-linux-distro) from [Adafruit](http://www.adafruit.com/). You should be able to install it fresh, follow the instructions to complete the setup, and then make sure ssh is turned on.

We are using a recent version of [OpenOCD](http://openocd.sourceforge.net/) that has gpio-bitbang capabilities. This has been added since their latest release (v0.6.1 as I write this), and the openocd package that apt-get will give you is even older (v0.5.x). This means we have to compile it from the source, but don't worry. I've figured out the steps.

First, get the dependencies, and then grab the code:

```bash
sudo apt-get update
sudo apt-get install -y autoconf libtool libftdi-dev
git clone --recursive git://git.code.sf.net/p/openocd/code openocd-git && cd openocd-git
````

This is the compiling as one massive command, copy and paste it in, and let it run. It'll take a while.

```bash
{
./bootstrap &&\
./configure --enable-sysfsgpio\
     --enable-maintainer-mode \
     --disable-werror \
     --enable-ft2232_libftdi \
     --enable-ep93xx \
     --enable-at91rm9200 \
     --enable-usbprog \
     --enable-presto_libftdi \
     --enable-jlink \
     --enable-vsllink \
     --enable-rlink \
     --enable-arm-jtag-ew \
     --enable-dummy \
     --enable-buspirate \
     --enable-ulink \
     --enable-presto_libftdi \
     --enable-usb_blaster_libftdi \
     --enable-ft2232_libftdi\
     --prefix=/usr\
&&\
make
} > openocd_build.log 2>&1
```

*Note:* You might get an error about not having makeinfo. It's not a problem.

Now, to actually install OpenOCD (You'll probably be asked for your password for any command that starts with `sudo`):

```bash
sudo make install
sudo cp -r tcl/ /usr/share/openocd
cd /usr/share/openocd/board
sudo wget https://gist.github.com/giseburt/e832ed40e3c77fcf7533/raw/e8c71233970e4d42eed7c3bf4b13390cdcf2a1fd/raspberrypi-due.tcl
```

##Wiring it up:

TODO
Cheap version: Using [this schematic of the Arduino Due](http://arduino.cc/en/uploads/Main/arduino-Due-schematic.pdf) and [this reference image](http://learn.adafruit.com/assets/3059) of the [Pi's GPIO pins](http://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/the-gpio-connector), you'll connect these connections:

|Due|Pi|
========
|tck|11|
|tms|25|
|tdi|10|
|tdi|9|
|trst|7|


##Running it:

Test that it connects properly:

```bash
sudo openocd -s /usr/share/openocd -f board/raspberrypi-due.tcl
```

You should get something that looks like this:
```text
pi@raspberrypi:~$ sudo openocd -s /usr/share/openocd -f board/raspberrypi-due.tcl
Open On-Chip Debugger 0.7.0-dev-00217-g74db7f9 (2013-04-03-02:00)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.sourceforge.net/doc/doxygen/bugs.html
Info : only one transport option; autoselect 'jtag'
SysfsGPIO nums: tck = 11, tms = 25, tdi = 10, tdi = 9
SysfsGPIO num: trst = 7
adapter speed: 500 kHz
adapter_nsrst_delay: 100
jtag_ntrst_delay: 100
cortex_m3 reset_config sysresetreq
Info : SysfsGPIO JTAG bitbang driver
Info : This adapter doesn't support configurable speed
1Info : JTAG tap: sam3.cpu tap/device found: 0x4ba00477 (mfg: 0x23b, part: 0xba00, ver: 0x4)
Info : sam3.cpu: hardware has 6 breakpoints, 4 watchpoints
```
