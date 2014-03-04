This page is just some notes for now but will ultimately turn into the jump page for testing a v9 with the test jig and also as a self-test w/o a test jig.

### Setup


### Tests

	Test # | Test | Summary Description
	-------|------|--------------------
	1 | Blue Light | On power-up the blue Power LED should light. This verifies 5v switcher operation and 3.3v LDO operation
	2 | Fan Test | On power-up the 12v fan output should power a 12v fan. This verifies 12v linear regulator
	3 | Programming | The board should be able to be programmed via the USB port using the BOSSA boot loader. On completion the RED indicator LED should flash slowly - approx once per second.
	2 | Basic Motion | Runs motors rapidly in both directions. Tests processor functionality as well as operation of step, direction, enable and Vref for all 5 axes. Requires two Dumb Steppers for 4th and 5th motors. Test invoked by `{test:100}`
	2 | Slow Motion | 
	2 | Red Light | On power-up 
