G2 is an Open Source CNC, 3D Printer, and general stepper-motor-based motion control firmware that runs on the next generation of TinyG hardware (currently in private beta) as well as the Arduino Due.

* [Hardware Needed To Use G2](#hardware-needed-to-use-g2)
* [Software Needed To Use G2](#software-needed-to-use-g2)

## Hardware Needed To Use G2

You will need whatever machine you are trying to control, such as a CNC machine, 3D printer, laser cutter, etc., along with the general knowledge of how to use that machine safely.

You will also need software to generate GCode. (TODO: Add a page with GCode software list.)

The following is a list of the minimum you will need strictly for the use of G2, regardless of what project you'll be using it for.

**Debugger/Programmer** (Not _strictly_ necessary, but very helpful) - [Atmel-ICE](http://www.atmel.com/tools/atatmel-ice.aspx).
* There are three ways they sell the Atmel-ICE: [Standard ($85)](http://store.atmel.com/PartDetail.aspx?q=p:10500375#tc:description), [Basic ($49)](http://store.atmel.com/PartDetail.aspx?q=p:10500377#tc:description) ([Digi-Key has them for $55](http://www.digikey.com/product-search/en?x=0&y=0&lang=en&site=us&KeyWords=atmel-ice-basic)), and [PCBA ($32)](http://store.atmel.com/PartDetail.aspx?q=p:10500376#tc:description). The standard comes with an adapter that you won't need for this project, and the PCBA is missing the $14 cable you _will_ need for the project, so the Basic is just right.
* This is not the Atmel _SAM-ICE_ or the Atmel _JTAG-ICE_, but just the _Atmel-ICE_. The Atmel SAM-ICE will work as well, but is older and more expensive, but has the advantage of being a (Atmel-limited) Segger J-Link. We will document here how to use the Atmel-ICE.
* You _can_ program the Due and the TinyG v9 boards without a debugger, but you cannot debug them.

**Compatible Target Board** - [Arduino Due](http://arduino.cc/en/Main/arduinoBoardDue) ($50) ([Adafruit](http://www.adafruit.com/products/1076), [Maker Shed](http://www.makershed.com/Arduino_Due_p/mksp16.htm), [Sparkfun](https://www.sparkfun.com/products/11589), [Digi-Key](https://www.digikey.com/product-highlights/us/en/arduino-arduino-due-board/2831), [Mouser](http://www.mouser.com/new/arduino/arduino-due/), [Newark](http://www.newark.com/arduino/a000062/dev-brd-sam3x8e-arm-cortex-m3/dp/47W2961))
* Currently the only publicly-available board G2 works on is the Arduino Due.

**Adapter Cabling/Shield** - gShield ($50) ([Synthetos Store](https://synthetos.myshopify.com/products/gshield-v5), [Adafruit](http://www.adafruit.com/products/1750), [Inventables](https://www.inventables.com/technologies/gshield)), or roll-your-own cabling.
* Depending on the project and your level of expertise, you may be able to wire stepper drivers you already have (as long as they have STEP/DIRECTION/ENABLE pinouts) to the Due or roll your own shield.
• The gShield provides three stepper drivers and pinouts for spindle controls and limit switches.

## Software Needed To Use G2

If you are _not_ planning on compiling the firmware yourself, then all you need the the Arduino 1.5+ IDE installed on your machine (for the `bossac` binary). You can then program any provided hex directly onto the Due or TinyG v9.

* [[Flashing G2 with Windows]]
* [[Flashing G2 with OSX]]


[![XKCD Comic About Compiling](http://imgs.xkcd.com/comics/compiling.png)](http://xkcd.com/303/)

If you _are_ planning on compiling the firmware yourself, go to the appropriate pages for your OS:

* [[Compiling G2 on Windows (Atmel Studio 6.2)]]
* [[Compiling G2 on OS X (with Xcode)]]
* Linux and OS X (using the command line) _TODO_