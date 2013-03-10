Instructions to set up Arduino Due in Atmel Studio6

* Clone the Arduino github

* Checkout lib-1.5 branch

* Create an empty Studio6 C/C++ ARM project.
 * Do not use the ASF templates
 * Set the initial file as main.cpp (this will be replaced by the Arduino main.cpp)
 * Select ATSAM3X8E chip and finish
 * Compile it. You should see `cmsis` in the project directory. You should also see a Debug directory created by AS6

* In the project directory create and populate these sub-directories using the github project sources. Studio6 is particular - use the Solution Explorer to first create New directory, then move the files into it and add them as Existing Items.  
 * `arduino` from `/Arduino/hardware/arduino/sam/cores/arduino`
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