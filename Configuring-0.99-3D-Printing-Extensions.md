_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

**!!!! TAKE NOTE: The 3D printer extensions documented on this page are pre-release and experimental (hence the leading underscore). These settings will NOT be present in the release ultimate 3D printing releases. Please expect any UIs or code written to these specifications to require changes. The settings on this page will be removed and will not be backwards compatible. !!!!**

# Heater Groups

This page documents axis settings. Each axis object ("group") has a collection of parameters for that axis. There are 6 axis groups, X,Y,Z linear axes, and A,B,C rotary axes. Not all axes have all parameters.

- Axis examples use X axis, but any axis is OK unless otherwise noted

## Summary
There are 3 heater groups. The examples below show heater group 1 - `{_he1:n}`

	Setting | Description | Notes
	--------|-------------|-------
	[{_he1e:...}](#_he1e-heater-enable) | Heater Enable | 0=off, 1=on 
	[{_he1p:...}](#_he1p-i-d-heater-pid-values) | Heater P (PID) | read/write 
	[{_he1i:...}](#_he1p-i-d-heater-pid-values) | Heater I (PID) | read/write 
	[{_he1d:...}](#_he1p-i-d-heater-pid-values) | Heater D (PID) | read/write 
	[{_he1st:...}](#_he1st---set-temperature) | Set setpoint temperature | write-only
	[{_he1t:...}](#_he1t---get-temperature) | Current temperature | read-only
	[{_he1op:...}](#_he1op---heater-output) | Heater PWM level | read-only
	[{_he1tr:...}](#_he1tr---thermistor-resistance) | Thermistor resistance | read-only
	[{_he1at:...}](#_he1at---at-temperature) | "At temperature" flag | read-only 
	[{_he1an:...}](#_he1an---heater-adc) | Heater ADC reading | read-only
	[{_he1fp:...}](#_he1fp---fan-power) | Fan power | read/write
	[{_he1fm:...}](#_he1fm---fan-minimum-power) | Fan minimum power | read/write
	[{_he1fl:...}](#_he1fl---fan-low-temperature) | Fan low temperature | read/write
	[{_he1fh:...}](#_he1fh---fan-high-temperature) | Fan high temperature | read/write

### {_he1:{fp:..., fm:..., fl:..., fh:...}} Fan Controls

### {_he1:{e:...}} Heater Enable

### {_he1:{p:..., i:..., d:...}} Heater PID Values

PID settings for heater feedback loop. Typical default values are:
```
For an extruder heater, 12volt:
  DEFAULT_P    7.0
  DEFAULT_I    0.05
  DEFAULT_D    150.0

For a heated bed:
  DEFAULT_P    9.0
  DEFAULT_I    0.12
  DEFAULT_D    400.0
```
