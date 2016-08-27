This page explains how to add a new `CONFIG` to the g2core project. Please read Adding and Revising Boards first if you haven't already:

1. [Adding and Revising Boards](Adding-and-Revising-Boards) _(please read this first)_
1. Adding and Revising Configurations and Settings _(this page)_
1. [Adding and Revising Motate Pinouts](Adding-and-Revising-Motate-Pinouts)
1. [Adding Configurations to an IDE](Adding-Configurations-to-an-IDE)

# Settings Files

The default settings file `settings_default.h` is the "base" settings that will always get applied. The default settings sets rational values for most configurations, and disables the motors axes, extruders, and other devices. Machine-specific settings files, like `settings_Printrbot_Simple1608.h` override these settings on an item-by-item basis. Any settings that are not `#define`d in the machine settings file will use the value in the default file. (*Note that this changed in build 098.04. Prior to that each settings file had to provide every possible setting.*)

In order to make a new settings file, you can copy one that's close to the configuration you want, or you can copy just the keys from the `settings_default.h` file that you wish to override.

# Adding a New Configuration

Configurations live in `boards.mk`, and are simply assigning a `BOARD` and `SETTINGS_FILE` based on the value of `CONFIG`. Here is the complete definition for `CONFIG=PrintrbotSimple1608` as an example:

```c++
ifeq ("$(CONFIG)","PrintrbotSimple1608")
    ifeq ("$(BOARD)","NONE")
        BOARD=printrboardG2v3
    endif
    SETTINGS_FILE="settings_printrbot_simple_1608.h"
endif
```

A few notes from that example:

1. It checks for the exact value of `CONFIG` to match.
1. It then *only* sets `BOARD` if it hasn't been set. (Before `boards.mk` is included, `BOARD` is defaulted to `"NONE"`.)
1. Then `SETTINGS_FILE` is set explicitly. We could also check to see if it has been set, but generally you use a `CONFIG` to set the `SETTINGS_FILE`, and wouldn't wish to override it.

One additional note: The same of the `CONFIG` is used in the output path, and should be something that can be a directory name without issues. Specifically, you shouldn't use spaces, slashes, etc. that could cause trouble with shell scripts, Makefiles, or C++ code. Numbers, upper and lowercase letters, dashes (`-`), dots (`.`), and underscores (`_`) are safe. Everything else should be avoided.