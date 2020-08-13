G2core is an Open Source CNC, 3D Printer, and general stepper-motor-based motion control firmware that runs on the next generation of g2core hardware as well as the Arduino Due.

* [Hardware Needed To Use g2core](#hardware-needed-to-use-g2gore)
* [Software Needed To Use g2core](#software-needed-to-use-g2core)
* [Developing on g2core](Getting-Started-with-g2core-Development)

## Hardware Needed To Use g2core
You will need whatever machine you are trying to control, such as a CNC machine, 3D printer, laser cutter, etc., along with the general knowledge of how to use that machine safely.

You will also need software to generate GCode.

The following is a list of the minimum you will need strictly for g2core, regardless of what project you'll be using it for.

**Compatible Target Board** - [Arduino Due](http://arduino.cc/en/Main/arduinoBoardDue) (~$50) ([Digi-Key](https://www.digikey.com/product-highlights/us/en/arduino-arduino-due-board/2831), [Newark](http://www.newark.com/arduino/a000062/dev-brd-sam3x8e-arm-cortex-m3/dp/47W2961), [Sparkfun](https://www.sparkfun.com/products/11589), [Maker Shed](http://www.makershed.com/Arduino_Due_p/mksp16.htm))
* Currently the only publicly-available board g2core works on is the Arduino Due.
* We plan to release the gQuadratic and gQuintic boards in fall 2016 that will also run g2core.

**Adapter Cabling/Shield**

If you're only going to use 3 axes (XYZ), the most popular shield is the gShield ($50) ([Synthetos Store](https://synthetos.myshopify.com/products/gshield-v5), [Adafruit](http://www.adafruit.com/products/1750), [Inventables](https://www.inventables.com/technologies/gshield)), or roll-your-own cabling.
* The gShield provides three stepper drivers and pinouts for spindle controls and limit switches.
* Depending on the project and your level of expertise, you may be able to wire stepper drivers you already have (as long as they have STEP/DIRECTION/ENABLE pinouts) to the Due or roll your own shield.

For using more axes (eg 4th, 5th, or 6th axes), the most popular options are the DJuke shields.  There are two to choose from, depending on your needs:
* [g2core shield (onboard drivers)](https://webshop.djuke.nl/kits/g2shield-onboard-kit) - Choose this one if you're using smaller stepper motor drivers. eg [Pololu](https://www.pololu.com/category/120/stepper-motor-drivers)
* [g2core shield (external drivers)](https://webshop.djuke.nl/kits/g2shield-external-kit) - Choose this one if your using external stepper motor drivers. eg [DM542](http://leadshine.com/series.aspx?type=products&category=stepper-products&producttype=stepper-drives&subtype=general-stepper-drives&series=dm)

## For developers (only)

For doing development of g2core itself, you'll need a hardware debugger.  People just wanting to *use* g2core and not do development can completely skip this bit. :smile:

**Debugger/Programmer** - Atmel-ICE debugger on [Mouser](http://www.mouser.com/ProductDetail/Atmel/ATATMEL-ICE-BASIC/?qs=sGAEpiMZZMsn4IaorHFpMNdmy%252bJMuxsJtWHi7YhUN7M%3d) and  [Digikey](http://www.digikey.com/product-detail/en/atmel/ATATMEL-ICE-BASIC/ATATMEL-ICE-BASIC-ND/4753381). 
* You _can_ program the Due and the TinyG v9 boards without a debugger, but to do realtime debugging you need an ICE of some sort. 
* Note that the Atmel-ICE is not the Atmel _SAM-ICE_ or the Atmel _JTAG-ICE_. The Atmel SAM-ICE will work as well, but is older and more expensive. All the examples on this site use the Atmel-ICE.

## Software Needed To Use g2core

If you are _not_ planning on compiling the firmware yourself, then all you need is the Arduino 1.5+ IDE installed on your machine (for the `bossac` binary). You can then program any provided hex directly onto the Due or TinyG v9.

* [Flashing g2core with Windows](Flashing-g2core-with-Windows)
* [Flashing g2core with OSX](Flashing-g2core-with-OSX)
* [Flashing g2core with Linux](Flashing-g2core-with-Linux)


