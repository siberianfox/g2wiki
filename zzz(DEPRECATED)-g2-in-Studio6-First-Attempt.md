**This is old and no longer valid.**

## Setting Up a Arduino Due Project in Studio6
There are some [links on the web](http://omarfrancisco.com/arduino-programing-using-atmel-studio-6-0/) explaining how to import the Arduino Due libraries into Studio6. That is not what this is. The instructions here are how to set up the Arduino Due 1.5 SAM sources as part of an Atmel Studio6 project. They are compiled and part of your project - not as a library.

* First ensure that your VM is adequately configured. Make sure .NET is up-to-date with Windows updates, and that you have at least 2.5 Gb RAM allocated for the VM. It's also a good idea to make sure you have at least 2 GB free on your VM's C: drive. See this [Atmel Studio 6.0 Readme](http://www.atmel.com/images/as6installer-6.0.1843-readme.pdf) or this [Atmel Studio 6.1 Beta Readme](http://www.atmel.com/Images/AtmelStudio-6.1.2440-beta-Readme.pdf)

* Clone the Arduino github

* Checkout lib-1.5 branch

* Create an empty Studio6 C/C++ ARM project.
 * Do not use the ASF templates
 * Set the initial file as main.cpp (this will be replaced by the Arduino main.cpp)
 * Select ATSAM3X8E chip and finish
 * Compile it. You should see `cmsis` in the project directory. You should also see a Debug directory created by AS6

* Create and populate these sub-directories in the project directory using the indicated github project sources. Studio6 is particular - use the Solution Explorer to first create New directory, then move the files into it and add them as Existing Items.  
 * `arduino` subdirectory with files from `/Arduino/hardware/arduino/sam/cores/arduino`
 * `system` from `/Arduino/hardware/arduino/sam/system`
 * `system\include` from `/Arduino/hardware/arduino/sam/system/include` 
 * `system\source` from `/Arduino/hardware/arduino/sam/system/source`
 * `variants` from `/Arduino/hardware/arduino/sam/variants` The variants dir only needs variant.h, variant.cpp, pins_arduino.h.  It does not need all the other stuff that's in and under the source directory
 * `motate` - create the directory - copy motate sources or leave this empty for now

* Add the following paths as `Relative Paths` to both the C and C++ compiler sections. _NOTE: At some point the Add Item and other dialog buttons disappeared from Visual Studio so I was not able to add these from the UI. I had to go into the .cppproj file and edit the XML directly_
<pre>
..\arduino
..\motate
..\cmsis
..\system
..\system\include
..\system\source
..\variants
</pre>

* Add the following the C++ Miscellaneous flags
<pre>
-fpermissive
</pre>

* Edit the chip.h file. The above almost worked w/o modifications. But the adc.h file has no definitions for the SAM3X family, of which the SAM3X8E on the Due is a member. You must redefine the SAM3XA_SERIES to contain the SAM3X8E (sam.h, line 88). Otherwise, it errs out in the first use of adc.h. I think in the Arduino IDE this is injected at compile time. The modifications to chip.h are below.
<pre>
/* The following modifies the SAM3XA_SERIES #defines from the sam.h file to fix compilation problems
   Ref: http://asf.atmel.no/docs/latest/common.services.calendar.example2.stk600-rcuc3d/html/group__sam__part__macros__group.html
*/
//#define SAM3XA_SERIES (SAM3A4 || SAM3A8)	// original define in sam.h file
#undef SAM3XA_SERIES
#define SAM3XA_SERIES (SAM3X4 || SAM3X8 || SAM3A4 || SAM3A8)
</pre>

* You will also need to add (or comment out) setup() and loop() functions in main.cpp. Here's the code for the classic LED blinky test:
<pre>
int led = 13;
void setup() {
    pinMode(led, OUTPUT);
}
void loop() {
    digitalWrite(led, HIGH);   // turn the LED on (HIGH is the voltage level)
    delay(500);               // wait 500 milliseconds
    digitalWrite(led, LOW);    // turn the LED off by making the voltage LOW
    delay(500);
}
</pre>

### Studio 6.1 Beta
We switched over from Studio6.0 (1996) to Studio 6.1 (2440-beta). Are we happy about his? Mostly. I've had a few cases where the SAM-ICE didn't want to go into debug mode (programming only, it says) and the .cppproj files are incompatible, making git maintenance and collaboration more complicated, but otherwise it's been OK.

## Using the Atmel SAM-ICE Debugger
The SAM-ICE (SAM In Circuit Emulator) is only officially supported programmer/debugger for the SAM3X8E under Studio6. It's a rebranded Segger J-Link, which is really a very nice unit, and the worth the $100, easy. Supposedly there are cheaper alternatives out like clones and some others, but I haven't tested these. Her's a useful [Arduino forum post about this](http://arduino.cc/forum/index.php?topic=134907).

### Installing the SAM-ICE
FYI: I use a VMware Windows XP image under OSX, so some instructions are applicable to my setup and can be ignored if they don't apply to you.

1. Get/buy a [SAM-ICE](http://www.mouser.com/ProductDetail/Atmel/AT91SAM-ICE/?qs=%2fha2pyFaduiSLswSzKKMtBJl3sH6Dl%252bGzJZewU2eQFM%3d) and the [Olimex JTAG adapter cable](http://www.mouser.com/Search/ProductDetail.aspx?R=ARM-JTAG-20-10virtualkey63420000virtualkey909-ARM-JTAG-20-10) from Mouser or similar. If you are like me this adds a week to your project and (at least) 2 sets of shipping charges. This can be avoided. Total cost will be about $125 including shipping. 

2. Plug in the ICE USB port. The VM should recognize it and bong at you. Go into the `Project/Properties` tab in AS6 and set the `Tool` to be the SAM ICE. Upgrade the ICE firmware if it asks you to. Set JTAG Clock to `Auto Clock`.

3. Plug the Olimex adapter cable into the ICE and onto the Due. The cable from the connector on the Due should exit over the Due's USB port - i.e. if you are looking the the Due and ATMEL reads right side up, the cable exits to the right of the board. Someone at Arduino put the tiny JTAG connector too close to the 4 pin 0.100 header and it doesn't seat fully, but it still makes good enough contact.

4. Go to the `Tools/Device Programming` tab and select the SAM-ICE. Set the device to be `ATSAM3X8E`. Select `JTAG` interface and hit `Apply`. Hit `Read`. You should get a device signature back and 3.3 volts.

5. Program the test code. Go to the `Memories` menu.

### SAM-ICE Debugger Notes and Caveats
I'm not used to HW debuggers as all the tinyg xmega code was debugged using the AVR simulator. Some of the caveats may be germane to all HW debuggers, some only to the SAM-ICE. I wouldn't know. But each of these caused some mystery or lost time which others might be able to avoid.
* There's a little toolbar tab for the ICE which is the Segger native program. Nice.
* If you upgrade to AS6.1 the Segger firmware will need to be updated. Once you do this the ICE does not appear to work properly on 6.0 any more. AT least that was my experience.
* The processor keeps running even when you (the debugger) are not. The cycles and stopwatch may continue to increment even if you are sitting on a breakpoint.
* Timers still run when you are on a breakpoint. If the counter value appears to flop around at random that's because effectively it is. You won't see the cycle counter and stopwatch increment cleanly when counting out a - say - 20 microsecond delay. If you enter an interrupt from this timer you won;t see the stopwatch tell you it took 20 uSec, it will tell you whatever it wants to, instead.
* Write-only registers such as REG_TC1_IER0 don't display what you just wrote to them in the IO view or memory. That's because they are write only (doh). 

#### Error message "None on SAM-ICE (000028011884) does not support debugging..."
This happens when the ICE doesn't know what device it's debugging. The "None" is the device name. [Clear as mud?](http://www.avrfreaks.net/index.php?name=PNphpBB2&file=printview&t=125017&start=0) It's as if AS6 lost it's mind. You can reset the device by clicking the Device button on the toolbar (which should actually say ATSAM3X8E - or whatever you are running) and resetting the device to be the same device.

#### SAM-ICE won't debug symbolically
[Here](https://github.com/synthetos/g2/wiki/SAM-ICE-Cannot-Debug)