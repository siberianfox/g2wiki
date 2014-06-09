Welcome to the g2 wiki!

This wiki documents the TinyG ARM port, which we are calling G2. G2 is pre-release as not all of [TinyG's](https://github.com/synthetos/TinyG) features are in it yet, but we have been working on it for over a year now and it's pretty stable for what it does do.

This wiki serves as a user and programmer manual and documents progress on the project. We (Synthetos) maintain the wiki, but it's an open wiki. If you want to post wiki-type stuff feel free to do it here. Please let us know via a github Issue if it's anything that needs active attention - Issues are good for requested changes, discussions and bona-fide software bugs.

![TinyG2 on an Arduino Due](http://farm4.staticflickr.com/3739/10301325295_31cb0dc6ab_h.jpg)

##What is G2?
G2 is a cross-platform ARM Port of the [TinyG](https://github.com/synthetos/TinyG) motion control system that runs on the [Arduino Due](http://arduino.cc/en/Main/ArduinoBoardDue) and on Synthetos hardware. On the Due it can be used with the [gShield](https://github.com/synthetos/grblShield/wiki) to build a high performance 3 axis motion control system. This wiki covers both platforms.

### Features
G2 has a number of advanced features, including:

* Full 6 axis motion control - XYX linear axes and ABC rotary axes
* Step outputs available for 6 motors (motors are mappable to axes)
* Jerk controlled motion for acceleration planning (S curve 3rd order motion planning)
* Extremely stable and jitter-free 200 Khz step generation
* RESTful interface using JSON over USB
* USB is native on the ARM chip and runs 12 Mbps or 480 Mbps
* Complete status and system state displays
* Advanced hardware abstraction layer for easy port to multiple ARM and non-ARM processing environments

###Some Clarification on Naming
It was pretty simple when there was just TinyG, but now there are more options. Here's the scheme:
 * TinyG is the Synthetos hardware. TinyG v8 is the Xmega based hardware, and v9 is the (first) ARM based hardware. Presumably there will be more TinyG variant, including a 5 axis board, expandable Kinen boards and some custom boards.
 * G2 is the ARM code base. It is cross platform based on the Motate hardware abstraction layer. G2 is written in C++ and is licensed under [GPLv2 with the BeRTOS exception](https://github.com/synthetos/g2/wiki/Licensing). We are now making the distinction between the firmware and the hardware as there are a growing number of combinations. 
 * G1. I guess that makes the original Xmega code G1. The G1 firmware still goes by TinyG on github. We have been merging the G1 and G2 code bases as you can see by examining the git branches and the code. The plan is to eventually retire G1 and run G2 on the Xmega hardware to continue support for the v8 and earlier TinyG hardware.  

#G2 User Pages
By and large TinyG2 works identically to TinyG, and most configuration and other questions are handled at the [TinyG wiki](https://github.com/synthetos/TinyG/wiki). Here are some pages that are specific to TinyG2. 
* [Project Status Page](https://github.com/synthetos/g2/wiki/G2-Project-Status-Page) along with differences between TinyG and G2

#TinyG2 Developer Pages
* [Development Environments](https://github.com/synthetos/g2/wiki/Development-Environments)
* [Programming TinyG2 onto the Due](https://github.com/synthetos/g2/wiki/Programming-TinyG2)
* [Makefile Notes](https://github.com/synthetos/g2/wiki/Makefile-Notes)
* [Atmel Studio6 Setup and Notes](https://github.com/synthetos/g2/wiki/g2-in-Studio6)
* [Programming the v9 board with Studio6 and a SAM-ICE](https://github.com/synthetos/g2/wiki/Programming-v9-with-Studio6-and-the-SAM-ICE)
* [Useful stuff to know](https://github.com/synthetos/g2/wiki/Useful-Stuff)
* [Licensing](https://github.com/synthetos/g2/wiki/Licensing)