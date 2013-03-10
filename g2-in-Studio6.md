Instructions to set up Arduino Due in Atmel Studio6

* Clone the Arduino github

* Checkout lib-1.5 branch 

* Create an empty Studio6 C/C++ ARM project.
 * Do not use the ASF templates
 * Select ATSAM3X8E
 * Set the initial file as main.cpp (this will be replaced by the Arduino main.cpp)
 * Compile it. You should see `cmsis` in the project directory
 * You should also see a Debug directory created by AS6

* In the project directory populate these sub-directories using the github project sources. 
 * `arduino` from `/Arduino/hardware/arduino/sam/cores/arduino`
 * `system` from `/Arduino/hardware/arduino/sam/system`
 * `system\include` from `/Arduino/hardware/arduino/sam/system/include` 
 * `system\source` from `/Arduino/hardware/arduino/sam/system/source`
 * `variants` from `/Arduino/hardware/arduino/sam/variants`

The variants dir only needs variant.h, variant.cpp, pins_arduino.h.  It does not need all the other stuff that's in and under the source directory

* Add the following `relative` paths to both the C and C++ compiler sections
<pre>
..\arduino
..\system
..\system\include
..\system\source
..\variants
</pre>

Here's where I'm a bit stuck. Looks like the adc.h file has no definitions for the SAM3X family, of which the SAM3X8E on the Due is a member. If you redefine the SAM3XA_SERIES to contain the SAM3X8E (sam.h, line 88) then it compiles. Otherwise, it errs out in the first use of adc.h.

You will also need to comment out the setup(), init() and loop() functions in main.cpp as these are undefined.