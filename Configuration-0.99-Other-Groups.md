_[<<< Back to Configuration for Version 0.99 Main Page](Configuration-for-Firmware-Version-0.99)_

##PWM Group (Pulse Width Modulation)
There is currently only one PWM channel (p1), but the configs are structured for multiple PWM groups. The PWM channel is set up to act as a remote control Electronic Speed Controller (ESC), but can be used for other PWM functions using these settings. 

	Setting | Description | Notes
	--------|-------------|-------
	{p1frq:_} | Frequency | in Hz, e.g. 100
	{p1csl:_} | Clockwise speed low | In RPM - arbitrary units unless you calibrate it, e.g. 1000
	{p1csh:_} | Clockwise speed high | In RPM
	{p1cpl:_} | Clockwise phase low | 0.000 to 1.000, e.g. 0.125 for 12.5% phase angle
	{p1cph:_} | Clockwise_phase_high | 0.000 to 1.000
	{p1wsl:_} | CCW speed low | In RPM 
	{p1wsh:_} | CCW speed high | In RPM
	{p1wpl:_} | CCW phase low | 0.000 to 1.000
	{p1wph:_} | CCW phase high | 0.000 to 1.000
	{p1pof:_} | Phase off | 0.000 to 1.000 used to set OFF phase for PWM devices that are not off at 0 phase