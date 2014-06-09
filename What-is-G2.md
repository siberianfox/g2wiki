##What is G2?
G2 is a cross-platform ARM Port of the [TinyG](https://github.com/synthetos/TinyG) motion control system that runs on the [Arduino Due](http://arduino.cc/en/Main/ArduinoBoardDue) and on Synthetos hardware. On the Due it can be used with the [gShield](https://github.com/synthetos/grblShield/wiki) to build a high performance 3 axis motion control system. This wiki covers both platforms.

### Features
G2 has a number of advanced features including:

* Full 6 axis motion control - XYX linear axes and ABC rotary axes
* Step outputs available for 6 motors (motors are mappable to axes)
* Jerk controlled motion for acceleration planning (S curve 3rd order motion planning)
* Extremely stable and jitter-free 200 Khz step generation
* RESTful interface using JSON over USB
* USB is native on the ARM chip and runs 12 Mbps or 480 Mbps
* Complete status and system state displays
* Advanced hardware abstraction layer for easy port to multiple ARM and non-ARM processing environments

###Some Clarification on Naming
It was pretty simple when there was just TinyG but now there are more options. Here's the scheme:
 * TinyG is the Synthetos hardware. TinyG v8 is the Xmega based hardware, and v9 is the (first) ARM based hardware. Presumably there will be more TinyG variants including a 5 axis board, expandable Kinen boards and some custom boards.
 * G2 is the ARM code base. It is cross platform based on the Motate hardware abstraction layer. G2 is written in C++ and is licensed under [GPLv2 with the BeRTOS exception](https://github.com/synthetos/g2/wiki/Licensing). We are now making the distinction between the firmware and the hardware as there are a growing number of combinations. 
 * G1. I guess that makes the original Xmega code G1. The G1 firmware still goes by TinyG on github. We have been merging the G1 and G2 code bases as you can see by examining the git branches and the code. The plan is to eventually retire G1 and run G2 on the Xmega hardware to continue support for the v8 and earlier TinyG hardware. 