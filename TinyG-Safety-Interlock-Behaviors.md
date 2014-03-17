This page describes the TinyG safety interlock shutdown system. The safety interlock is designed to shut down a machine in a controlled manner if a safety envelope switch is tripped. Machine state is preserved so that jobs can be resumed after a safe conditions are restored.

*** Note - At the current time this page is just a specification. No TinyG's in production have this feature ***

##Physical and Electrical Components

The TinyG safety interlock is comprised of the following hardware / electrical components.

* Interlock switch loop. One or more normally closed switches (NC) may be wired in series. If any switch is tripped - opening the loop - a safety interlock signal is generated. This signal starts the interlock delay timer. It also causes a CPU interrupt, signaling that the CPU should execute a controlled shutdown before the timer expires.

* Interlock delay timer. An isolated circuit located on the TinyG board delays by a pre-set time once the interlock switch loop is triggered. The timer is an analog circuit that uses no programming or other software and therefore does not require software validation.

* System hardware lockouts. Once the timer expires it activates a set of hardware lockouts located on the TinyG board. All lockouts are implemented in hardware - no software is involved in their operation. Lockouts are:
 * All stepper motor step signals are driven low through a dedicated line buffer. This prevents the processor from moving the stepp 
 * The Spindle ON signal is driven to logic LOW (disabled)

##Definition of Events

The following events can occur as part of a safety shutdown. 


On Sat, Mar 15, 2014 at 12:22 PM, Alden Hart <alden@tenmilesquare.com> wrote:
Mike,

When the safety interlock cuts out what behaviors should occur? Define a number of behaviors :

(1) Interlock tripped (broken)

- immediately an interrupt on the ARM triggers. this tells the processor it needs hustle and shutdown the spindle and the motors.
- at the same time a one shot timer on the board starts. it does not disable anything, just starts a count down
 
(2) Safety timer times out

- when the once shot timer completes, the disable lines for the step trains are disabled. the motor will still maintain a holding torque, so it will not loose position, but the processor will be unable to move it. a disable signal is sent to the speed controller, and the PWM wave is disabled. (why both? there are two ways spindles get controlled... depending on spindle controller type one needs a "chip enable" type signal, and the other needs its PWM disabled.)
 
(3) Interlock re-engaged

- the timer is cancelled and reset, and the interrupt line to the ARM returns to enabled. the step and direction lines are re-enabled, and the spindle PWM and spindle enabled signals are restored. to prevent a race condition, the processor should debounce the interlock interrupt and wait for it to stabilize before restoring motion. 
 
(4) manual or automatic cycle-start to resume program, operation

this is machine dependent, but i would favor manual resume. the state machine should probably be left in a FEED_HOLD state.
 
Stepper behavior
On (1) the program goes into a feedhold. We need to know a worst case for the OM to go into feedhold so we can pick a time constant for the timer.

it will vary, but it will most certainly be less than 1 second at the most. it has to do with the time it takes to decelerate the X/Y & Z carriages at maximum speed (1500mm/min). 
 
On (2) the step lines are disengaged (overridden) by the interlock timer

they're tied to what ever their last value is. if they are high they remain high, if they are low, they remain low.
 
On (3) the step lines are re-engaged.

they ripple the current input to the output. if the processor changed the input while the timer fired then it is a software bug. to detect this state, the oneshot firing could trigger a second interrupt, but i think it will be unnecessary in non-debugging situations.
 
On (4) does the program start from a new cycle start? Or does it wait for the user to manually issue the cycle start?

it would need to do the ramp up to finish what ever line it was in the middle of. i would think that waiting for user input to continue would be prudent.
 
Spindle behavior
On (1) the CPU turns the spindle control to OFF and drops PWM to 0 RPM (but NOT off - need to feed the ESC)

yup.
 
On (2) the spindle ON line is overridden to be OFF (same as disconnecting the step line). This is more complicated because the notion of active HI or active LO must now be baked into the interlock circuit. It cannot be under SW control.

true. always use active high for safety circuits, that way, if the control signal is broken it defaults to disabled instead of enabled (no interlock). by the same token, always use NC switches, so if they're improperly wired, the machine is disabled.
 
On (3) at a HW level the Spindle ON line is re-engaged.
 
On (4) what happens? As above, does the program resume it's previous state or wait for a manual cycle start? How about the PWM? Does to go back to the RPM that it was before the break?

at this point, the processor should have returned the spindle control PWM cycles to 0 RPM. re-engaging should warm up the spindle until it comes up to the desired speed, and then continue as before.
 

As you can see - we need some detailed review of behaviors.

hope my answers clear up some of the details. FYI: i think we're going to design our own speed controller for certification purposes. i'm going to shoot for a low cost closed loop speed controller ;)

--mikest