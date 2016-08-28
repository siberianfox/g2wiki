
##Commands and Reports
These $configs invoke reports and functions

	Command | Description | Notes
	--------|-------------|-------
	[{sr:n}](#sr---status-report) | get Status Report | SR also sets status report format in JSON mode
	[{qr:n}](#qr---queue-report) | get Queue Report | 
	[{qf:_}](#qf---queue-flush) | flush_planner_queue | Used with '!' feedhold for jogging, probes and other sequences. Usage: {qf:1}
	[{md:n}](Power-Management) | Disable motors | Unpower all motors
	[{me:_}](Power-Management) | Energize motors | Energize all motors with timeout in seconds 
	[{test:_}](#test---run-self-test) | Invoke self tests | $test=n for test number; $test returns help screen in text mode
	[{defa:1}](#defa---reset-default-profile-settings) | reset to DEFAults | Reset to compiled default settings
	[{flash:1}] | Enter_FLASH_loader | WILL ERASE AND REFLASH BOARD. BE CAREFUL WITH THIS
        ^x | Reset tinyG | cntl+x restarts tinyG same as hardware rest button
	[{help:_}] | Show help screen | Show system help screen; $h also works

Note: Status report parameters is settable in JSON only - see JSON mode for details
<br>
