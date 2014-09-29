1) Install hombrew: http://brew.sh/
   Look for "Install Homebrew” near the bottom of that page. Follow those instructions.

2) In terminal, run:
`brew install openocd`

3) cd into the <project>/TinyG2/ directory, have the V9 all wired up and run:
`make VERBOSE=2 PLATFORM=G2v9i debug`

That should have you debugging in gdb.

Info here: https://sourceware.org/gdb/onlinedocs/gdb/index.html#Top
