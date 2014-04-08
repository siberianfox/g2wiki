This page describes the TinyG CNC controller's safety interlock shutdown system. The safety interlock is designed to shut down a machine in a controlled manner if a safety envelope switch is tripped. Machine state is preserved so that jobs can be resumed after a safe conditions are restored.

**Note - At the current time this page is just a specification. No TinyG's in production have this feature**

##Hardware Components

The controller safety interlock is comprised of the following hardware components.

* **Interlock switch loop**. One or more normally closed switches (NC) may be wired in series. If any switch is tripped - opening the loop - a safety interlock signal `Interlock_NC` is driven to logic HI.

* **Interlock delay timer**. An isolated circuit located on the controller board delays by a pre-set time once Interlock_NC is activated. The timer is an analog circuit that uses no programming or other software and therefore does not require software validation.

* **Hardware lockouts**. Once the timer expires it activates a set of hardware lockouts located on the controller board. All lockouts are implemented in hardware - no software is involved in their operation. Lockouts are:
 * All four stepper motor step signals are driven to logic LO through a line buffer dedicated to this purpose. This prevents the controller from moving the stepper motors until the lockout is disabled.
 * The Spindle ON signal is disabled via a line buffer by driving it logic LO
 * The Spindle pulse-width-module (PWM) signal is disabled via a line buffer driving it to logic LO

* **Interlock header**
The interlock mechanism may also need to control external controllers such as spindles, coolant systems & laser drivers. To support this the interlock pinouts should be included on the Kinen & SPI control interfaces. There are two output logic level signals:
 * Interlock triggered. This is the signal that should be connected to the MCU interrupt and is enabled when the lockout switch loop opens.
 * Interlock timed out. This is the signal that is active when the interlock has timeout has passed, and is connected to the lockout circuitry.

##Sequence of Events

	Step | Sequence | Description
	------|------------|---------|-------------
	0 | **Power Up** | The controller will not start operation unless the `Interlock_NC` signal is LO (switches closed)
	1 | **Interlock Opens** | Opening one or more interlock switches causes `Interlock_NC` to go HI (active). The following things happen simultaneously:
	1a | **Controller Alarm** | The controller immediately enters an alarm state during which no command input is honored or executed.
	1b | **Controller Feedhold** | The controller performs a feedhold to stop the current movement while preserving positional information. A feedhold from 1500 mm/min with a jerk value of 500 million mm/min^3 will execute in approximately 210 milliseconds. 
	1c | **Spindle Stop** | The controller stops the spindle if the spindle is running. The spindle should decelerate to a stop in less than _____ seconds
	1d | **Timer Start** | The interlock delay timer is started to time an interval of approximately 500 milliseconds. 
	2 | **Timer Expires** | Once the interlock delay timer expires the hardware lockouts described above are activated, as described following:
	2a | **Stepper Motor Lockout** | The stepper motor lockout circuit disables step signals to the stepper motor drivers. This causes all motors to stop if they have not already been stopped by the controller. Motors still maintain a holding torque, so the do not lose position, but the controller is will now be unable to move them.
	2b | **Spindle Lockout** | Disable signals are sent to the spindle lockout circuits. The Spindle ON signal is forced LO (inactive), and the Spindle PWM is disables. This is because there are two ways spindles get controlled, depending on spindle controller type one needs a "chip enable" type signal, and the other needs its PWM disabled. Note that the spindle must always be active HI in a safety situation, so that loss of signal (i.e. Spindle ON going LO) will not enable the spindle. 
	3 | **Interlock_NC_Restored** | At some point after 1 or 2 (above) the interlock switches may be restored. This re-enables the hardware and allows the controller to receive commands. To prevent a race condition, the processor debounces the interlock interrupt and waits for it to stabilize before restoring motion.
	4 | **Job Resume** | The user program can issue a `cycle start` to resume the job that was halted. This should be a manual operation requiring user interaction. This part of the sequence is under the control of the CNC application program. The controller requires the following sequence to be sent: (1) re-start the spindle to the RPM desired, (2) issue a cycle start to resume the job. Depending on the nature of the CNC job the CNC application may need to command the controller to perform other recovery actions.


Notes for Mike:

* You said regarding the stepper driver lockout: "they're tied to what ever their last value is. if they are high they remain high, if they are low, they remain low." Im not sure how this works. If the buffers go into tri-state then it might preserve the step pulse phase, but I will need to put a pulldown on this line. The only time that this matters is if the feedhold did not fully take effect. That is the only time that the step pulse might actually be high. In that case you have already lost position - as the feedhold did not complete before the timer fired. Otherwise the step pulse will always be low.
 

* The spindle behavior we describe will kill the ESC as the PWM is shut down. Are you sure this is what we want?
"On (1) the CPU turns the spindle control to OFF and drops PWM to 0 RPM (but NOT off - need to feed the ESC)

yup."