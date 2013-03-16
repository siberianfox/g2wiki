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

Here's where I'm a bit stuck. Looks like the adc.h file has no definitions for the SAM3X family, of which the SAM3X8E on the Due is a member. If you redefine the SAM3XA_SERIES to contain the SAM3X8E (sam.h, line 88) then it compiles. Otherwise, it errs out in the first use of adc.h.

You will also need to comment out the setup() and loop() functions in main.cpp as these are undefined, or add these functions in main or somewhere accessible to main:

<pre>
void setup( void )
{
	
}

void loop( void )
{
	
}
</pre>

## Using the Atmel SAM-ICE Debugger
The SAM-ICE (SAM In Circuit Emulator) is only officially supported programmer/debugger for the SAM3X8E under Studio6. It's a rebranded Segger J-Link, which is really a very nice unit, and the worth the $100, easy.
 
1. Get/buy a [SAM-ICE](http://www.mouser.com/ProductDetail/Atmel/AT91SAM-ICE/?qs=%2fha2pyFaduiSLswSzKKMtBJl3sH6Dl%252bGzJZewU2eQFM%3d) and the [Olimex JTAG adapter cable](http://www.mouser.com/Search/ProductDetail.aspx?R=ARM-JTAG-20-10virtualkey63420000virtualkey909-ARM-JTAG-20-10) from Mouser or similar. If you are like me this adds a week to your project and (at least) 2 sets of shipping charges. This can be avoided. Total cost will be about $125 including shipping. 