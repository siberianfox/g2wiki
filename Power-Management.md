Power management is used to keep the steppers on when you need them and turn them off when you don't. 

Stepper motors consume maximum power when idle. They hold torque and get hot. If you shut off power the motor has (almost) no holding torque. Most lead screw or geared machines will hold position if you shut off the power when the motor is not moving, but belt machines generally do not. 

In either case it's usually a good idea to keep all motors powered during a machining cycle to maintain absolute machine position. Depending on your machine you either may want to remove power some number of seconds after the machining cycle, or leave the motors powered on for as long as the machine is on.

You also generally want to use power management to de-power the machine if it's left unattended for an extended time. You don't want to leave the steppers on for extended idle times such as walking away from your machine and leaving it on overnight with the motors idling. 

The power management commands let you set up the right set of actions for your machine and use.

## Global Power Management Commands
These commands affect all motors.
 
	Setting | Description | Notes
	--------|-------------|-----------------------------
	[{me:...}](#me-motor-enable) | Enable motors | {md:n} will enable all motors for default timeout (mt). Set value to seconds to override default timeout value
	[{md:...}](#md-motor-disable) | Disable motors | {md:n} will disable all motors. Set 1-6 to disable motor N only
	[{mt:...}](#mt-motor-timeout) | Set motor enable timeout | In seconds, up to 1 million seconds
	$pwr | Report enable states | PWR returns all motor enables states. This is like a virtual motor LED that returns 1 if motor N is enabled (LED is lit), 0 if not
	$pwr1 | Report enable state | PWRn returns a single motor state

<pre>
Text mode examples:
$me=180    Lock all motors for 3 minutes for a tooling operation 
$md        Disable all motors
$mt=300    Set timeout to 5 minutes
$pwr       Query power enabled for all motors
$pwr2      Query power enabled for motor 2

JSON mode equivalents:
{me:180}
{me:n}
{mt:300}
{pwr:n}
{pwr2:n}
</pre>

###{me:...} Motor Enable
This will turn on any motor that is not disabled ($1pm=0). If provided a valid motor number it will enable that motor only. 

###{md:...} Motor Disable
This will turn off any motor that is not permanently enabled ($1pm=1). If provided a valid motor number it will disable that motor only.

###{mt:...} Motor Timeout
Sets the number of seconds before a motor will shut off automatically. When the timeout starts is set by context:

$1pm=0 Disabled motors are always off. They do not time out
$1pm=1 Always-on motors are always on. They do not time out, either
$1pm=2 Motors that are powered-in-cycle begin timeout at the end of the cycle, which is when the last motor stops moving.
$1pm=3 Motors that are powered-while-moving begin timeout at the end of their movement

Motor timeouts are suspended during feedholds. This allows changing or adjusting tools without loss of position.

##Per-Motor Commands

	Setting | Description | Notes
	--------|-------------|-----------------------------
	{1:{pm:n}} | Display power mode | Returns one of the power modes below
	[{1:{pm:0}}](#1pm0-motor-disabled) | Disabled | Motor will not run, and is not enabled by `{me:N}` 
	[{1:{pm:1}}](#1pm1-motor-always-powered) | Always powered | Motor always powered and is not disabled by `{md:t}` 
	[{1:{pm:2}}](#1pm2-motor-powered-in-cycle) | Powered in cycle | Motor is powered during machining cycle (any axis is moving) and for `{mt:N}` seconds after cycle stops
	[{1:{pm:3}}](#1pm3-motor-powered-when-moving) | Powered when moving | Motor is powered when it is moving and for `{mt:N}` seconds afterwards. Motors in this state can disable themselves during cycles if timeout is less than cycle time.

<pre>
JSON mode examples:
{1pm:0}  or {1:{pm:0}}
{2pm:1}  or {2:{pm:1}}
{1pm:2}  or {1:{pm:2}}
{4pm:3}  or {4:{pm:3}}

Text mode equivalents:
$1pm=0     Motor 1 disabled
$2pm=1     Motor 2 always powered
$1pm=2     Motor 1 powered during a machining cycle (any motor moving)
$4pm=3     Motor 4 only powered when it is moving
</pre>

These commands affect all motors and take effect as soon as they are issued.

###{1:{pm:0}} Motor Disabled
This will turn off motor power and prevent the motor from turning on. Disabling the motor will prevent that axis from participating in any move. The motor will not be affected by `{me:N}` or `{md:t}` commands. This setting takes effect immediately.

###{1:{pm:1}} Motor Always Powered
This will turn on motor power and leave it on until the board is shut down. The motor will not be affected by `{me:N}` or `{md:t}` commands. You generally do not want to use this mode as it will leave the motors on for extended periods of time if you do not power down the machine. Better to set a long motor power timeout and use Powered In Cycle (2)

###{1:{pm:2}} Motor Powered In Cycle
This will turn on the motor power at the start of any move on any axis (a "cycle"), and will de-energize the motors `{mt:N}` seconds after the cycle is complete (i.e. all motion stops). The timeout interval is set by the `{mt:N}` value (see [Global Power Management Commands](#global-power-management-commands)).

###{1:{pm:3}} Motor Powered When Moving
This will turn on the motor power only when that axis is moving, and will remove power `{mt:N}` seconds after that axis stops moving.

