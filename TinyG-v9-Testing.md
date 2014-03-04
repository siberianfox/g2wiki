This page is just some notes for now but will ultimately turn into the jump page for testing a v9 with the test jig and also as a self-test w/o a test jig.

### Required Equipment / Parts

	# | Item | Summary Description
	--|------|--------------------
	1 | TinyG v9 Board | Referred to as UUT for unit-under-test
	2 | Bench Power Supply | Bench power supply capable of 2 amps at 24 volts. In the absence of a bench supply a standard 24 VDC will suffice, but some test steps will not be possible
	3 | Stepper Motors | Five NEMA23 stepper motors with a current rating of approximately 2.5 to 3 amps
	4 | Dumb Stepper Fins | Two Synthetos DumbStepper fins. Used to test slots 4 and 5

### Setup

	# | Step | Summary Description
	--|------|--------------------
	1 | Wire and Verify Power | With the power supply On power-up the blue Power LED should light. This verifies 5v switcher operation and 

### Tests

	# | Test | Summary Description
	--|------|--------------------
	1 | Blue Light | On power-up the blue Power LED should light. This verifies 5v switcher operation and 3.3v LDO operation
	2 | Fan Test | On power-up the 12v fan output should power a 12v fan. This verifies 12v linear regulator
	3 | Programming | The board should be able to be programmed via the USB port using the BOSSA boot loader. On completion the RED indicator LED should flash slowly - approx once per second.
	4 | Basic Motion | `{test:100}` runs 5 motors in sequence at high speed in both directions. Tests processor functionality as well as operation of step, direction, enable and Vref for all 5 axes. Requires two Dumb Steppers for 4th and 5th motors.
	5 | Slow Motion | 
	6 | Red Light | 
