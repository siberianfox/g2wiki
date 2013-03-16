## Setting Up a Arduino Due Project in Studio6
There are some [links on the web](http://omarfrancisco.com/arduino-programing-using-atmel-studio-6-0/) explaining how to import the Arduino Due libraries into Studio6. That is not what this is. The instructions here are how to set up the Arduino Due 1.5 SAM sources as part of an Atmel Studio6 project. They are compiled and part of your project - not as a library.

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

* Add the following paths as `Relative Paths` to both the C and C++ compiler sections
<pre>
..\arduino
..\system
..\system\include
..\system\source
..\variants
</pre>

* Edit the sam.h file. The above almost worked w/o modifications. Looks like the adc.h file has no definitions for the SAM3X family, of which the SAM3X8E on the Due is a member. If you redefine the SAM3XA_SERIES to contain the SAM3X8E (sam.h, line 88) then it compiles. Otherwise, it errs out in the first use of adc.h. I think in the Arduino IDE this is injected at compile time.
<pre>
/* Entire SAM3XA series 
   Changed this define to fix compilation problems
   Ref: http://asf.atmel.no/docs/latest/common.services.calendar.example2.stk600-rcuc3d/html/group__sam__part__macros__group.html
   This file is found in: C:\Program Files\Atmel\Atmel Studio 6.0\extensions\Atmel\ARMGCC\3.3.1.128\ARMSupportFiles\Device\ATMEL\sam.h
   It has been included in the project in the Extras directory
*/
//#define SAM3XA_SERIES (SAM3A4 || SAM3A8)
#define SAM3XA_SERIES (SAM3X4 || SAM3X8 || SAM3A4 || SAM3A8)
</pre>

* You will also need to comment out the setup() and loop() functions in main.cpp as these are undefined, or add these functions in main or somewhere accessible to main:

<pre>
void setup( void )
{
	
}

void loop( void )
{
	
}
</pre>

## Using the Atmel SAM-ICE Debugger
The SAM-ICE (SAM In Circuit Emulator) is only officially supported programmer/debugger for the SAM3X8E under Studio6. It's a rebranded Segger J-Link, which is really a very nice unit, and the worth the $100, easy. Supposedly there are cheaper alternatives out like clones and some others, but I haven't tested these. Her's a useful [Arduino forum post about this](http://arduino.cc/forum/index.php?topic=134907).

FYI: I use a VMware Windows XP image under OSX, so some instructions are applicable to my setup and can be ignored if they don't apply to you.

1. Get/buy a [SAM-ICE](http://www.mouser.com/ProductDetail/Atmel/AT91SAM-ICE/?qs=%2fha2pyFaduiSLswSzKKMtBJl3sH6Dl%252bGzJZewU2eQFM%3d) and the [Olimex JTAG adapter cable](http://www.mouser.com/Search/ProductDetail.aspx?R=ARM-JTAG-20-10virtualkey63420000virtualkey909-ARM-JTAG-20-10) from Mouser or similar. If you are like me this adds a week to your project and (at least) 2 sets of shipping charges. This can be avoided. Total cost will be about $125 including shipping. 

2. Plug in the ICE USB port. The VM should recognize it and bong at you. Go into the `Project/Properties` tab in AS6 and set the `Tool` to be the SAM ICE. Upgrade the ICE firmware if it asks you to. Set JTAG Clock to `I don't know yet`.

3. Plug the Olimex adapter cable into the ICE and onto the Due. The cable from the connector on the Due should exit over the Due's USB port - i.e. if you are looking the the Due and ATMEL reads right side up, the cable exits to the right of the board. Someone at Arduino put the tiny JTAG connector too close to the 4 pin 0.100 header and it doesn't seat fully, but it still makes good enough contact.

4. Go to the `Tools/Device Programming` tab and select the SAM-ICE. Set the device to be `ATSAM3X8E`. Select `JTAG` interface and hit `Apply`. Hit `Read`. You should get a device signature back and 3.3 volts.

5. Program the test code. Go to the `Memories` menu.