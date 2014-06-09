# Compiling G2 from source

## Tools Needed To **Use* G2
_TODO: Move this section to a different page, but keep it._

**Debugger/Programmer** (Not _strictly_ necessary, but very helpful) - [Atmel-ICE](http://www.atmel.com/tools/atatmel-ice.aspx).
* There are three ways they sell the Atmel-ICE: [Standard ($85)](http://store.atmel.com/PartDetail.aspx?q=p:10500375#tc:description), [Basic ($49)](http://store.atmel.com/PartDetail.aspx?q=p:10500377#tc:description) ([Digi-Key has them for $55](http://www.digikey.com/product-search/en?x=0&y=0&lang=en&site=us&KeyWords=atmel-ice-basic)), and [PCBA ($32)](http://store.atmel.com/PartDetail.aspx?q=p:10500376#tc:description). The standard comes with an adapter that you won't need for this project, and the PCBA is missing the $14 cable you _will_ need for the project, so the Basic is just right.
* This is not the Atmel _SAM-ICE_ or the Atmel _JTAG-ICE_, but just the _Atmel-ICE_. The Atmel SAM-ICE will work as well, but is older and more expensive, but has the advantage of being a (Atmel-limited) Segger J-Link. We will document here how to use the Atmel-ICE.

**Compatible Target Board** - [Arduino Due](http://arduino.cc/en/Main/arduinoBoardDue) ($50) ([Adafruit](http://www.adafruit.com/products/1076), [Maker Shed](http://www.makershed.com/Arduino_Due_p/mksp16.htm), [Sparkfun](https://www.sparkfun.com/products/11589), [Digi-Key](https://www.digikey.com/product-highlights/us/en/arduino-arduino-due-board/2831), [Mouser](http://www.mouser.com/new/arduino/arduino-due/), [Newark](http://www.newark.com/arduino/a000062/dev-brd-sam3x8e-arm-cortex-m3/dp/47W2961))
* Currently the only publicly-available board G2 works on is the Arduino Due.

**Adapter Cabling/Shield** - gShield ($50) ([Synthetos Store](https://synthetos.myshopify.com/products/gshield-v5), [Adafruit](http://www.adafruit.com/products/1750), [Inventables](https://www.inventables.com/technologies/gshield)), or roll-your-own cabling.
* Depending on the project and your level of expertise, you may be able to wire stepper drivers you already have (as long as they have STEP/DIRECTION/ENABLE pinouts) to the Due or roll your own shield.
• The gShield provides three stepper drivers and pinouts for spindle controls and limit switches.

# Tools Needed To Build G2 From Source

## On OS X
* You will need XCode ([Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)) and/or the XCode Command-Line Tools.
* XCode installs the tools, but you have to run a separate installer to make them available to the terminal.
  * On Mavericks, it's as simple as `xcode-select --install` in terminal and following the prompts. You will not need XCode separately, but the G2 project does have a convenient XCode project that makes navigating and compiling the code much easier.


