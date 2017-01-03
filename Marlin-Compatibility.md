# Marlin Compatibility Mode

### Why have a Marlin Compatibility mode?

For 3D printing, there is a large number of utilities that are written to speak to [Marlin](https://github.com/MarlinFirmware/Marlin) or various other mostly-compatible variants. Even though there are several things that don't translate between Marlin and g2core, we can make an effort to at least accept gcode sliced for Marlin, and to be able to conditionally speak the Marlin protocol.

This gives the authors of those various utilities (assuming that they are still being maintained) some time to support the g2core protocol and gcode flavor.

### What it's not

This will support the most-used gcode and protocol of Marlin, and nothing more.

What we don't support as Marlin (you have to use the g2core protocol for these):

* Configuration of the machine and various parameters
* Setup of motors, heaters, etc.
* Tuning PID, limit switch polarity, etc.

## Communications protocol

There are a few concepts that convert more-or-less from g2core and Marlin:

| g2core | Marlin | Notes |
| :---: | :---: | --- |
| `{"r":...,"f":[...]}` | `ok` | In response to every sent gcode, g2core will return a response JSON, and Marlin will return `ok`. <br\> (Marlin has an ["advanced `ok`"](https://github.com/MarlinFirmware/Marlin/blob/7bea5e5e5701de0b90b6c422c954337ce860bb0f/Marlin/Marlin_main.cpp#L8704) option as well.) |
| `{"he1st":210}` | ?? | Set temperature of extruder 1 to `210`. |
| `{"he1st":n}` | ?? | Request the set temperature of extruder 1. |
| `Nxxx` (gcode) | `Nxxx`&nbsp;(gcode)&nbsp;`*`&nbsp;(checksum) | [Marlin supports a checksum, and has special requirements on line numbers.](http://reprap.org/wiki/G-code#Special_fields), where g2core accepts arbitrary line numbers and doesn't support a non-standard checksum. |

## Supported `Mxxx` codes

## Unsupported

