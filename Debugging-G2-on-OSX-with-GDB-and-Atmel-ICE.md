See also: [Compiling G2 on OSX with Xcode](Compiling-G2-on-OS-X-(with-Xcode))
## Hardware Required
You will need the following hw and configuration
* TinyGv9 board
* Atmel-ICE debugger
* Both plugged into an OSX machine via USB, preferably without a hub. See the [Apple] / About this Mac / More Info / System Report / USB section. Hit command R if you want to refresh. Some times getting these connected takes some unplugging and replugging to get USB to behave.

## Install OpenOCD Debugger
1. Install hombrew: http://brew.sh/
   Homebrew is a package manager for OSX which (in this case) provides an easy way to install OpenOCD. Look for "Install Homebrew” near the bottom of that page. Follow those instructions in Terminal.<br>
<pre>
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
</pre>

As instructed, run `brew doctor` before you install anything, and
install the Command Line Tools for Xcode: https://developer.apple.com/downloads. Search "Command Line Tools for Xcode". You may need to register as an Apple Developer for this step.
You can also run this command in terminal to install Xcode Command Line Tools:
```bash
xcode-select --install
```
...and select Install in the popup window.

2. In terminal, run:<br>
`brew install openocd`

3. cd into the <project>/TinyG2/ directory, have the V9 all wired up and run something like:<br>
`make VERBOSE=2 PLATFORM=G2v9k debug`<br>
...for your platform

You may see a message instructing you to copy openocd.cfg.example to openocd.cfg. Copy then edit this file for the debugger as per the instructions in the file.

That should have you debugging in gdb. If you get connection errors make sure the Atmel-ICE is connected as in Hardware, above.

On OSX it might pop a dialog box requesting you to accept an inbound connection. Do it. You may also need to try to connect more than once for it to work.

##Using GDB
* References:
  * [GDB reference manual](https://sourceware.org/gdb/onlinedocs/gdb/index.html#Top)
  * [GDB cheat sheet](http://users.ece.utexas.edu/~adnan/gdb-refcard.pdf)

* Some useful commands

  * The `make` command (make VERBOSE=2 PLATFORM=G2v9k debug) sets the "current elf" for your PLATFORM
  * `load` programs the board with the current elf for your PLATFORM. (Do this after a recompile)
  * `monitor reset halt` to reset the board. Do this every time you load.

* This is Embedded GDB, so `run` and some commands in the cheat sheets do not work

* for help on any command type `help <command>`

### A Very Brief Embedded GDB primer
Even if you are familiar with GDB, you may find a few important differences when using it with an embedded project. This will not be a thorough how-to on this topic, but will hopefully serve enough to get you started.

Using `make debug` will call `gdb` (technically it's `arm-none-eabi-gdb` or whatever variant is specified in the makefile for your board, but we'll just call it `gdb` from now on) with parameters and configuration to:
* Know which binary/elf file to use for this debug session.
* Connect to the board using `openocd`.
* And halt the board, and leave you at a command line prompt.

Now for a few commands to get you started:

#### Debugging and Loading
In order to debug you need a debugger capable of debugging the chip you use. For the Due and V9 we use either a Atmel SAM-ICE (which is a OEM Segger J-Link locked to Atmel ARMs) or the cheaper and more recently released Atmel ICE. Either can be found at Mouser.com.

You choose which debug adapter you wish to use by *copying* the `openocd.cfg.example` file to `openocd.cfg` and then editing the `openocd.cfg` file. Simply follow the instructions in that file. Hint: It's just uncommenting the correct lines.

Once that is configured, you'll need to make sure the debugger is correctly connected to the hardware and plugged into the computer, and all of the hardware is powered properly.

Then call `make debug` - don't forget to include any of the additional variables on your make command line, or it may build for the wrong platform. A few eaxmples:

```bash
# If you normally call this to build:
make PLATFORM=UltimakerTests SETTINGS_FILE=my_settings.h

# Then just add the "debug" to the end
make PLATFORM=UltimakerTests SETTINGS_FILE=my_settings.h debug
```

This will build the code (if it needs it) and then open `gdb` connected to the hardware and reset and halt the TinyG2 system.

#### Quitting the debugger

`q` or `quit` will exit GDB. If the processor is running, you may need to type `CTRL-C` to get the prompt first.

You will often need to reset or power-cycle the board after quitting GDB, since it will likely leave it in a halt state.

#### Flashing the firmware onto the board

`load` - Since `gdb` already knows what bianry/elf file to use, you simple have to type `load` at the `(gdb)` prompt. Here's an example of a succesful flash session:

```bash
(gdb) load
Loading section .text, size 0x1da10 lma 0x80000
Loading section .relocate, size 0x164 lma 0x9da10
Start address 0x82c88, load size 121716
Transfer rate: 14 KB/sec, 13524 bytes/write.
(gdb)
```

#### Resetting the processor

`monitor reset halt` - Once we flash new code, or if we just want the "program" to start over, we call `monitor reset halt` to reset and halt the processor. This will freeze the processor at the reset state, before any code has executed.

You *must* call `monitor reset halt` after (or *immediately* before) a `load` command so that the processor starts from the new code. Otherwise, the processor will likely crash, sometimes after appearing to work for some time. **To avoid nightmare debug sessions, be sure you always follow `load` with `monitor reset halt`!**

#### Running the code

Full docs are [here](https://sourceware.org/gdb/current/onlinedocs/gdb/Continuing-and-Stepping.html#Continuing-and-Stepping) for `continue`, `step`, and `next`.

`c` or `continue` - To resume the processor operating normally, you type `c` (or the full word `continue`) and hit return.

Once it's running, gdb will not show a prompt, and you cannot see any output from the processor. To pause the processor again, hit `CTRL-C`.

NOTE: If you are used to using gdb in a desktop OS, you will notice that we didn't call `run` -- in fact, it won't work. In embedding programming, you're driving the whole OS on the processor, not just running a single program.

#### Step to the next line (into or over functions)

`s` or `step` - Continue running your program until control reaches a different source line, then stop it and return control to GDB. This will step *into* functions, since that would change which line is being executed.

`n` or `next` - Continue to the next source line in the current (innermost) stack frame. This will skip over function calls, since that would change which stack frame is executing.

`fin` or `finish` - Continue running until just after function in the selected stack frame returns. Print the returned value (if any).

#### Getting a backtrace (listing the function call stack)

Full documentation is [here](https://sourceware.org/gdb/current/onlinedocs/gdb/Backtrace.html#Backtrace) for `bt` and related.

`bt` or `backtrace` - Print a backtrace of the entire stack: one line per frame for all frames in the stack. (Note that this may get LONG, and you can make it stop with `CTRL-C`.)

Example:
```gdb
(gdb) bt
#0  0x00085df2 in _controller_HSM () at ./controller.cpp:172
#1  controller_run () at ./controller.cpp:150
#2  0x000889fc in main () at ./main.cpp:169
```

Note that the current execution frame is on the top, and `main()` is on the bottom.

Interpretation: `main()` called `controller_run()`, which called `_controller_HSM()`, which is where we are "currently." It also tells us that that is specifically at line 172 of `./controller.cpp`.

These are more advanced topics covered very well all over the internet, so I won't go into detail here. I will list a few usefull commands to start of your research, however:

#### Breakpoints

Documented [here](https://sourceware.org/gdb/current/onlinedocs/gdb/Set-Breaks.html#Set-Breaks).

* `b` -- set a breakpoint. Will default to "here" but you can pass it a function name or a `filename:line` to break on a specific line.
* `info br` -- show breakpoints. In the listing the first column in the breakpoint number.
* `dis N` -- Disables (but does not delete) breakpoint number N.
* `ena N` -- Enables (but does not delete) breakpoint number N.
* `del N` -- Deletes (but does not delete) breakpoint number N.

#### Showing Variables

Documented [here](https://sourceware.org/gdb/current/onlinedocs/gdb/Variables.html#Variables)
* `p expression` - execute the given expression and print the results. Takes most C syntax (but not all) in the expression. **Warning!** It is a common mistake to accidentally change the state or variables when trying to display them. For example, this is a terrible way to test if the `should_blow_up` variable is true:
  ```gdb
	(gdb) p should_blow_up=1
	$1 = 1 '\001'
	```
	It will actually *set* `should_blow_up` to true!! A check, like normal C, would be with the double equals:
	```gdb
	(gdb) p should_blow_up==1
	$2 = false
	```
	`p` will also nicely print out entire structures. You may have to dereference pointers, however:
	```gdb
	(gdb) p mb.r
	$4 = (mpBuf_t *) 0x20071434 <mb+220>
	(gdb) p *mb.r
	$5 = {
		pv = 0x20071368 <mb+16>,
		nx = 0x20071500 <mb+424>,
		# ... clipped some for brevity ...
		jerk = 0,
		recip_jerk = 0,
		cbrt_jerk = 0,
		gm = {
			linenum = 0,
			motion_mode = 0 '\000',
			# ... clipped some for brevity ...
			spindle_mode = 0 '\000'
		}
	}
	(gdb)

	```

#### Fixing a board that won't boot after flashing

Sometimes the board will appear like it was flashed correctly, however it will never boot up. You may also notice that all LED's are on, and that only one USB port is listed when trying to connect. This may happen when you flash the chip that has never been programmed before, or the previous image somehow gets corrupted.

To fix this, after successful `load` command run:

`monitor at91sam3 gpnvm set 1`

and then

`monitor reset halt`

disconnect and reboot the board.

#### More Advanced Stuff

Try these commands:
* help disp
* help comm

Useful for displaying and tracking variables during a debugging run.

One option is to `disp`lay a variable or structure every time you step. Like the following:
`disp arc`

To delete type `undisplay arc`
