This page contains general information about TinyG v9 hardware. 

_PRELIMINARY - This is still being iterated_

TinyG v9 is intended for use by individuals building a system, as well as by system manufacturers looking to reduce costs in assembly and maintenance while maintaing high reliability. TinyG v9 has a number of features designed to support these 2 (sometime competing) goals. In general, there are choices for how to assemble and use the IO connectors.

### Power Connector

The power connector can be populated with either a standard 5.00 mm screw terminal block or a straight or right angle 2 position friction lock header. The friction lock header is rated for 7 amps, so if you are using NEMA23 motors (or larger) we recommend using the screw terminal option. 

Friction Lock Header (Molex parts are listed but any equivalent part should work)
* Molex 171813-0002     2 position right angle header
* Molex 171814-0002     2 position right angle header

Mating parts are:
* Molex 09-50-3021     2 position housing
* Molex 08-50-0134     crimp terminal (2 required per plug)

### Motor Connectors

The v9 has a dual hole patterns for motors. It will accommodate 3.81mm screw terminals but can also be populated for vertical or right angle 0.156 friction lock headers. 

Friction Lock Headers
* Molex 26-60-4040      4 position straight header
* Molex 26-60-5040      4 position right angle header

Mating parts are:
* Molex 09-50-3041     4 position housing
* Molex 08-50-0134     crimp terminal (4 required per plug)

### Limit Switch Connectors
The homing/limit switches are brought in on 4 position terminal blocks for X, Y and Z axes. The pinout is labeled on the bottom silkscreen and (from top to bottom) is: (1) ground, (2) 3.3v, (3) min in, (4) max in. The switch inputs are designed for standard "dry" switches but will accommodate opto or other active inputs as long as they do not exceed 3.3v. These signals are conditioned with and RC network (2.7K / 0.47 uF) and have ESD protection.

The X, Y, Z inputs are also available to a 2x13 dual inline 0.100" header. The header also provides  Amin, Amax and a safety interlock inputs.

### Spindle and Coolant Output Signals

These output signals are available on a terminal block and also on a 2x6 dual inline 0.100" header. These signals are buffered and may be driven at either 3.3v or 5v, depending on the setting of a jumper on the board.

* Spindle On/Off
* Spindle Direction
* Spindle PWM signal
* Coolant On/Off

### SPI Connector
SPI signals are available on a 2x5 dual inline 0.100" header. These are at 3.3v.
* MOSI
* MISO
* CLK
* Reset
* CS1
* CS2
* 3.3v
* GND
* Interlock IN
* Interlock OUT

### Stepper Breakout Connectors
Stepper signals are taken out on four 1x5 single inline 0.100" headers. These are at 3.3v. Please note that the pinout of these connectors is different than similar connectors on the v7 and v8 boards

* 3.3v
* Step
* Enable (active low)
* Direction
* Ground


