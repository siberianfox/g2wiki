#TinyG2 Project Status

### Main Differences 
For most practical purposes the G2 and TinyG functionality are the same. Differences are noted below.

Main differences between TinyG and G2
* G2 uses an ARM processor instead of the Atmel Xmega. G2 runs on the Atmel SAM8X3E (84 MHz) and other processors in that family. TinyG runs on an Atmel Xmega192 (32 MHz)
* G2 runs a step clock rate of 200 KHz as opposed to 50 KHz.
* G2 runs native USB as opposed to an FTDI USB adapter
* G2 provides digitally controllable current references as opposed to on-board potentiometers
* Both TinyG and G2 compute 6 axes. G2 can output 6 axes (depending on platform hardware), TinyG is limited to 4 axes

###Update as of June 9, 2014
We recommend working from the edge branch, which is the most stable branch to date. As changes are made and tested in lower branches these will be posted to edge.

###Update as of Oct 16, 2013
Build 020.20 has been pushed to edge and master. 

The following features are still in development for G2:
* Non-volatile storage (EEPROM). The Atmel SAM ARM chips do not have EEPROM. Non-volatile storage is not provided or must be provided off-board.
* Canned tests ($test=N) are not yet implemented on G2
