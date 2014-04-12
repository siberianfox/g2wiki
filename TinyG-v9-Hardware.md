This page contains general information about TinyG v9 hardware. The v9 is still under development and may change.

_PRELIMINARY - This is still being iterated_

TinyG v9 is intended for use by individuals building a system, as well as by system manufacturers looking to reduce costs in assembly and maintenance while maintaining high reliability. TinyG v9 has a number of features designed to support these 2 (sometime competing) goals. In general, there are choices for how to assemble and use the IO connectors.

### Board Dimensions

* Width  3.500" (x dimension)
* Height 4.000" (y dimension)
* Holes (x, y)
  * 0.150", 0.150"
  * 3.350", 0.150"
  * 3.350", 3.850"
  * 0.150", 3.850"

### Power Connector

The power connector can be populated with either a standard 5.00 mm screw terminal block or a straight or right angle 2 position 0.156" (3.96mm) friction lock header. The friction lock header is rated for 7 amps, so if you are using NEMA23 motors (or larger) we recommend using the screw terminal option. 

Friction Lock Header (Molex parts are listed but any equivalent part should work)
* Molex 171813-0002     2 position right angle header
* Molex 171814-0002     2 position right angle header

Mating parts are:
* Molex 09-50-3021     2 position housing
* Molex 08-50-0134     crimp terminal (2 required per plug)

Power inputs are listed below. Pin 1 is towards the inside of the board (closest to the big capacitor C1)

1. Ground
1. +Vmot (+12v = +30v)

### Motor Connectors

The v9 has a dual hole patterns for motors. It will accommodate 3.81mm screw terminals but can also be populated for vertical or right angle 0.156" (3.96mm) friction lock headers. 

Friction Lock Headers
* Molex 26-60-4040      4 position straight header
* Molex 26-60-5040      4 position right angle header

Mating parts are:
* Molex 09-50-3041     4 position housing
* Molex 08-50-0134     crimp terminal (4 required per plug)

Phase outputs are listed below. Pin 1 is towards the LED end of the board (bottom)

1. A1
1. A2
1. B2
1. B1

### Limit Switch Connectors
The homing/limit switches are brought in on 4 position terminal blocks for X, Y and Z axes. The pinout is listed below. Pin 1 is towards the power input side of the board (top).

1. Ground
1. +3.3v
1. Min in (X, Y or Z)
1. Max in (X, Y or Z)

The switch inputs are designed for standard "dry" switches but will accommodate opto or other active inputs as long as they do not exceed 3.3v. These signals are conditioned with an RC network (2.7K / 0.47 uF) and have ESD protection.

The X, Y, Z inputs are also available to a 2x13 dual inline 0.100" header. The header also provides  Amin, Amax and a safety interlock inputs. The header footprint is large enough to accommodate oversized headers such as box and latching headers.

### Spindle and Coolant Output Signals

These output signals are available on a terminal block and also on a 2x6 dual inline 0.100" header. The header footprint accommodates box and latching headers.

These signals are buffered and may be driven at either 3.3v or 5v, depending on the setting of a jumper on the board.

The pinout is listed below. Pin 1 is towards the power input side of the board (top).

1. Ground
1. +3.3v
1. Spindle On/Off
1. Spindle Direction
1. Spindle PWM signal
1. Coolant On/Off

### SPI Connector
SPI signals are available on a 2x5 dual inline 0.100" header. The header footprint accommodates box and latching headers. All signals are 3.3v. The pinout is listed below. Pin 1 is the square pin towards the power input side of the board (top). Pins alternate with odd pins on the left and even pins on the right.

1. MISO
1. 3.3v
1. CLK
1. MOSI
1. ~Reset (active low)
1. GND
1. CS1
1. CS2
1. Interlock IN
1. Interlock OUT

### Stepper Breakout Connectors
Stepper signals are taken out on four 1x5 single inline 0.100" headers. The header footprint accommodates box and latching headers. All signals are 3.3v. Please note that the pinout of the stepper breakout connectors is different than similar connectors on the v7 and v8 boards. Pin 1 is the pin closest to the power inout side of the board (top).

1. 3.3v
1. Direction
1. ~Enable (active low)
1. Step
1. Ground


