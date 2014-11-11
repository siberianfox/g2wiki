See also: [Compiling G2 on OSX with Xcode](Compiling-G2-on-OS-X-(with-Xcode))
###Hardware
You will need the following hw and configuration
* TinyGv9 board
* Atmel-ICE debugger
* Both plugged into an OSX machine via USB, preferably without a hub. See the [Apple] / About this Mac / More Info / System Report / USB section. Hit command R if you want to refresh. Some times getting these connected takes some unplugging and replugging to get USB to behave.

###Install
1. Install hombrew: http://brew.sh/
   Look for "Install Homebrew” near the bottom of that page. Follow those instructions in Terminal.<br>
<pre>
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
</pre>
As instructed, run `brew doctor` before you install anything, and 
install the Command Line Tools for Xcode: https://developer.apple.com/downloads. Search "Command Line Tools for Xcode". You may need to register as an Apple Developer for this step.
You can also run this command in terminal to install Xcode Command Line Tools:
`xcode-select --install` (and select Install in the popup window).

2. In terminal, run:<br>
`brew install openocd`

3. cd into the <project>/TinyG2/ directory, have the V9 all wired up and run:<br>
`make VERBOSE=2 PLATFORM=G2v9i debug`

That should have you debugging in gdb. If you get connection errors make sure the Atmel-ICE is connected as in Hardware, above.

####Use GSB
* References: 
  * [GDB reference manual](https://sourceware.org/gdb/onlinedocs/gdb/index.html#Top)
  * [GDB cheat sheet](http://users.ece.utexas.edu/~adnan/gdb-refcard.pdf)

* Some useful commands
  * `monitor reset halt` to reset
  * `monitor load` programs the board with the current elf

* The `make` command sets the "current elf"

* This is Embedded GDB, so some commands in the cheat sheets do not work:
  * `run`
