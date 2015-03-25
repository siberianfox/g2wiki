- go to the motate_pin_assignments.h file, e.g. 
in the v9_3x8c/motate_pin_assignments.h file
you need to swap the pin you want to "steal from" with the debug pin you want
Say spindle:
Change
pin_number kSpindle_EnablePinNumber         = 112;

to

pin_number kSpindle_EnablePinNumber         = -1;

- assign one or more debug pins to pins you wantot "take over". Like 112

pin_number kSpindle_EnablePinNumber         = -1; // 112


And make sure to assign the original functions to -1!
Then at the top of the file you want to instrument, put (or, more likely, uncomment) the OutputPin definition
for example, at the top of plan_exec.cpp now is:
//OutputPin<kDebug1_PinNumber> exec_debug_pin1;
//OutputPin<kDebug2_PinNumber> exec_debug_pin2;
OutputPin<kDebug3_PinNumber> exec_debug_pin3;
//OutputPin<-1> exec_debug_pin3;

then to set the pin:
exec_debug_pin3 = 1;

and the clear it:
exec_debug_pin3 = 0;

don't commit the changed pin assignments
only the pin changes in the code itself
