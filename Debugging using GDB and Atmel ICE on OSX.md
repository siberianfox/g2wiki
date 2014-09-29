1) Install hombrew: http://brew.sh/
   Look for "Install Homebrew” near the bottom of that page. Follow those instructions.<br>
<pre>
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
Install the Command Line Tools for Xcode: https://developer.apple.com/downloads
Run `brew doctor` before you install anything
Run `brew help` to get started
</pre>

2) In terminal, run:<br>
`brew install openocd`

3) cd into the <project>/TinyG2/ directory, have the V9 all wired up and run:<br>
`make VERBOSE=2 PLATFORM=G2v9i debug`

That should have you debugging in gdb.

Info here: https://sourceware.org/gdb/onlinedocs/gdb/index.html#Top
