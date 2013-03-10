Instructions to set up Arduino Due in Atmel Studio6

* Clone the Arduino github
* Checkout lib-1.5 branch 
* Create an empty Studio6 C/C++ ARM project. 
 * Select ATSAM3X8E. Do not use the ASF templates.
 * Set the initial file as main.cpp (this will be replaced by the Arduino main.cpp)
 * Compile it. You should see `cmsis` in the project directory
 * You should also see a Debug directory created by AS6
* In the project directory populate these sub-directories using the github project sources. 
 * `arduino` from `/Arduino/hardware/arduino/sam/cores/arduino`
 * `system` from `/Arduino/hardware/arduino/sam/system`
   * `system\include` from `/Arduino/hardware/arduino/sam/system/include` 
   * `system\source` from `/Arduino/hardware/arduino/sam/system/source` 