Welcome to the g2 wiki!

This wiki documents the TinyG ARM port, which (at least for now) we are calling TinyG2. TinyG2 is really in alpha as not all of [TinyG's](https://github.com/synthetos/TinyG) features are in it yet, but we have been banging on it for a few months now and it's pretty stable for what it does do. (See this page for differences)

This wiki serves as a user and programmer manual, and documents progress on the project. We (Synthetos) maintain it, but it's an open wiki. If you want to post wiki-type stuff, feel free to do it here. Please let us know via a github Issue if it's anything that needs active attention - Issues are good for requested changes, discussions and bona-fide software bugs.

![TinyG2 on an Arduino Due](http://farm4.staticflickr.com/3739/10301325295_31cb0dc6ab_h.jpg)

#What is TinyG2?
TinyG2 is a cross-platform ARM Port of the [TinyG](https://github.com/synthetos/TinyG) motion control system that runs on the [Arduino Due](http://arduino.cc/en/Main/ArduinoBoardDue). It can be used with the [gShield](https://github.com/synthetos/grblShield/wiki) to build a high performance 3 axis motion control system.

G2 has a number of advanced features, including:

* Full 6 axis motion control - XYX linear axes and ABC rotary axes
* Step outputs available for 6 motors (motors are mappable to axes)
* Jerk controlled motion for acceleration planning (S curve 3rd order motion planning)
* RESTful interface using JSON
* Extremely stable and jitter-free 100 Khz step generation
* Complete status and system state displays

#TinyG2 User Pages
By and large TinyG2 works identically to TinyG, and most configuration and other questions are handled at the [TinyG wiki](https://github.com/synthetos/TinyG/wiki). Here are some pages that are specific to TinyG2. 
* [Project Status Page](https://github.com/synthetos/g2/wiki/G2-Project-Status-Page) along with differences between TinyG and G2
* [Programming TinyG2 onto the Due](https://github.com/synthetos/g2/wiki/Programming-TinyG2)

#TinyG2 Developer Pages
* [Development Environments](https://github.com/synthetos/g2/wiki/Development-Environments)
* [Makefile Notes](https://github.com/synthetos/g2/wiki/Makefile-Notes)
* [Atmel Studio6 Setup and Notes](https://github.com/synthetos/g2/wiki/g2-in-Studio6)
* [Useful stuff to know](https://github.com/synthetos/g2/wiki/Useful-Stuff)
* [Licensing](https://github.com/synthetos/g2/wiki/Licensing)