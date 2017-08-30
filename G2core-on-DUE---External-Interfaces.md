# Driving Steppers with G2core running on the Arduino DUE

THIS PAGE IS UNDER CONSTRUCTION
Send Questions to carl.mcgrath@synthetos.com
## Overview
The Arduino DUE development platform is ideal for getting started with G2core and for prototyping multi-axis projects.
G2core builds for CONFIG=gShield will build firmware and set up pinouts for the DUE supporting 6 Axes, X,Y,Z,A,B and C plus outputs for SpinPWM , all 6 Axis limit switches and a few other I/Os. 

Here are some useful references
* https://github.com/synthetos/g2/wiki/Arduino-DUE-Pinout-for-g2core
* https://github.com/grbl/grbl/wiki/Connecting-Grbl

### A Quick 3 axis solution
Use a gShield on the DUE with 3 on-board 8818 driver devices for 3 Axis operation
* A good reference: https://github.com/synthetos/grblShield/wiki/Using-grblShield
### External interfaces for full 6 axis operation and more powerful stepper drivers
The diagram below is hosted at https://github.com/cmcgrath5035/G2core-DUE-External-Interfaces
Included are suggested interface devices (proto board, etc) to level shift and buffer the DUE outputs.
Consider this a template, there are numerous similar devices that could be used, the MOSFETS are rather inexpensive and easy to work with

![Schematic Page](https://github.com/cmcgrath5035/G2core-DUE-External-Interfaces/blob/master/G2core%20DUE%20External%20Interfaces.png)

A downloadable PDF is also available: https://github.com/cmcgrath5035/G2core-DUE-External-Interfaces/blob/master/G2core%20DUE%20External%20Interfaces.pdf

Here is a list of external drivers reported to work with the DUE and G2core.
These are just examples, not recommendations
Feel free to edit this page with your experienced results
* DM542 https://www.omc-stepperonline.com/stepper-motor-driver/leadshine-dm542-digital-stepper-drive-20-50-vdc-with-10-42a-dm542.html
* w.i.p.