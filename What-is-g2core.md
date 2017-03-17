G2core is a cross-platform ARM Port of the [TinyG](https://github.com/synthetos/TinyG) motion control system that runs on ARM 32bit processors such as the [Arduino Due](http://arduino.cc/en/Main/ArduinoBoardDue) and the Synthetos v9 hardware. 

On the Due it can be used with the [gShield](https://github.com/synthetos/grblShield/wiki) to build a high performance 6 axis motion control system. This wiki covers both platforms.

### Features
g2core has a number of advanced features including:

* Full 6 axis motion control - XYZ linear axes and ABC rotary axes
* Step outputs available for 6 motors (motors are mappable to axes)
* Jerk controlled motion for acceleration planning (S curve 3rd order motion planning)
* Extremely stable and jitter-free 200 kHz step generation
* RESTful interface using JSON over USB
* USB is native on the ARM chip and runs 12 Mbps or 480 Mbps
* Complete status and system state displays
* Advanced hardware abstraction layer for easy port to multiple ARM and non-ARM processing environments

### Some Clarification on Naming
It was pretty simple when there was just TinyG but now there are more options. Here's the scheme:
 * TinyG is the Synthetos hardware. TinyG v8 is the Xmega based hardware, and v9 is the (first) ARM based hardware. Presumably there will be more TinyG variants including a 5 axis board and a number of g2core-based custom boards such as the Shopbot sbv300 and the Printrbot printrBoardG2.
 * g2core is the ARM code base. It is cross platform based on the Motate hardware abstraction layer. G2core is written in C++ and is licensed under [GPLv2 with the BeRTOS exception](https://github.com/synthetos/g2/wiki/Licensing). We are now making the distinction between the firmware and the hardware as there are a growing number of combinations. 
 * The original TinyG firmware still goes by TinyG on github. We have been merging the TinyG and g2core code bases as you can see by examining the git branches and the code. Many new features in TinyG have originated in g2core. The original plan was to eventually retire the TinyG code base and run g2core on the Xmega hardware. But g2 core has evolved past the point that the 8-bit Xmega can run it. So we will continue support the v8 TinyG hardware and will be adding new features, but it will not run the g2core firmware. The main distinction will be in other types of machines. TinyG will continue to operate as a CNC milling controller and most features relevant to that application will be ported in. However, new features such as supporting 3d printing, laser cutters and other applications will not be.