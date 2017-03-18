This page is just some notes for now but will ultimately turn into the jump page for testing a v9 with the test jig and also as a self-test w/o a test jig.

### Required Equipment / Parts

No. | Item | Summary Description
----|------|--------------------
1 | TinyG v9 Board | Referred to as UUT for unit-under-test
2 | Bench_Power_Supply | Bench power supply capable of 2 amps at 24 volts. In the absence of a bench supply a standard 24 VDC will suffice, but some test steps will not be possible
3 | Stepper Motors | Five NEMA23 stepper motors with a current rating of approximately 2.5 to 3 amps. If the UUT has quick connect motor terminals motors should be terminated with <fill in the parts here>
4 | Dumb Stepper Fins | Two Synthetos DumbStepper fins. Used to test slots 4 and 5
5 | Volt Meter | A volt meter is used to verify proper operation of the power supply

### Setup

No. | Step | Summary Description
----|------|--------------------
1 | Wire_and_Verify_Power | First, verify that the power supply is delivering 24v and observes polarity - i.e. ground is really ground and 24 is really 24v. Then with the power supply off, connect the 24 volt power supply to the power terminals. Verify polarity (double check) before turning on power.
2 | Wire Motors | Wire motors to motor terminals 1 through 5. Depending on the board construction this will require either quick connect terminals or screw terminals
3 | Plug in DumbSteppers | ...to slots 4 and 5. See picture for proper orientation



### Tests

No. | Test | Summary Description
----|------|--------------------
1 | Blue Light | On power-up the blue Power LED should light. This verifies 5v switcher operation and 3.3v LDO operation
2 | Fan Test | On power-up the 12v fan output should power a 12v fan. This verifies 12v linear regulator
3 | Programming | The board should be able to be programmed via the USB port using the BOSSA boot loader. On completion the RED indicator LED should flash slowly - approximately once per second.
4 | Basic Motion | `{test:100}` runs 5 motors in sequence at high speed in both directions. Tests processor functionality as well as operation of step, direction, enable and Vref for all 5 axes. Requires two Dumb Steppers for 4th and 5th motors.
5 | Slow Motion |
6 | Red Light |
