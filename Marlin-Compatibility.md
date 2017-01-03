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

Also, this marlin support is a temporary option to ease transition, and will be deprecated once it becomes too difficult to support. This depreciation, once it has been scheduled, will be announced in the g2core changelog and on social media.

## Communications protocol and gcode handling

Gcode on Marlin must be all uppercase. g2core accepts both upper- and lower-case mixed.

On Marlin, motion of the extruder is a mixture of `E` and `T` words, where on g2core `A`, `B`, and `C` are currently used. (This is planned to change to using spindle-based controls.)

On g2core, `G0` does not accept an `F` word and `G1` requires one (but it is 'sticky,' so it only be included once, and from then on when it needs changed).

There are a few other concepts that convert more-or-less from g2core and Marlin:

| g2core | Marlin | Notes |
| :---: | :---: | --- |
| `{"r":...,"f":[...]}` | `ok` | In response to every sent gcode, g2core will return a response JSON, and Marlin will return `ok`. <br\> (Marlin has an ["advanced `ok`"](https://github.com/MarlinFirmware/Marlin/blob/7bea5e5e5701de0b90b6c422c954337ce860bb0f/Marlin/Marlin_main.cpp#L8704) option as well.) |
| `{"he1st":210}` | `M104 S201 T1` | Set temperature of extruder 1 to `210`. |
| `{"he1st":n}` | `M105` | Request the set temperature of extruder 1 (g2core) or current extruder and bed (Marlin). |
| `Nxxx` (gcode) | `Nxxx`&nbsp;(gcode)&nbsp;`*`&nbsp;(checksum) | [Marlin supports a checksum, and has special requirements on line numbers](http://reprap.org/wiki/G-code#Special_fields), where g2core accepts arbitrary line numbers and doesn't support a non-standard checksum. |
| `{"out4":0.5}` | `M106 S128` | Set output on the fan (or generic PWM output `4` for g2core) to 50%. |


## Supported `Mxxx` codes

- [ ] **`M104`, `M109`, `M140`, `M190`, `M108` - temperature control**
  - On Marlin, you set the extruder temperature (and return immediately) with a `M104 Snnn Tnnn`
    - `Snnn` indicates the temperature in celsius
    - `Tnnn` (optional) indicates the tool, with the first being `0`
    - If you wish to wait until the extruder is at temperature, change `M104` to `M109`.
      - Cancel the wait with `M108` (with no further words).
      - With `Sxxx` it will only wait for heating. Change the `Sxxx` to `Rxxx` to wait for cooling or heating.
      - Both `M104` and `M109` (with either `S` or `R`) will set the target temperature.
      - During heat-up, the temperature of the extruder and heat-bed will be automatically printed every second.
  - On Marlin, the bed temperature is set with `M140 Snnn`, and it does NOT wait for the bed to heat up.
    - `Snnn` indicates the temperature in celsius
    - Wait for the heat bed with `M190 Sxxx` (for heating only) or `M190 Rxxx` (for both heating and cooling)
      - Like `M109`, `M190` will also set the target and print the temperature of the extruder and bed every second until while it's waiting.
      - `M190` is also cancelled with `M108`.

- [ ] **`M105` - temperature polling**
  - Response is in the format: `ok T:18.2 /0.0 B:16.6 /0.0 @:0 B@:0`
    - `T:nnn` contains the temperature of the "current" extruder (in celsius).
    - ` /nnn` contains the set temperature of the given extruder (in celsius).
    - `B:nnn` contains the bed temperature (in celsius).
    - ` /nnn` after that is the set temperature of the bed (in celsius).
    - `@:nnn` is the power output level (0-PID_MAX, which appears to generally be 255).
    - `B@:nnn` is the bed power output level (0-MAX_BED_POWER, which appears to generally be 255).
  - For Ultimaker Marlin, this differs from the response format of `M109` and `M190` of `T:101.9 E:0 W:1`
    - `T:nnn` is the same
    - `E:nnn` is the [extruder number](https://github.com/Ultimaker/Ultimaker2Marlin/blob/master/Marlin/Marlin_main.cpp#L1427)
    - `W:nnn` is the seconds of wait time left (and may be `?`) for the temperature to be considered "stable".
  - For stock Marlin, the output format of `M109` and `M190` is the same as `M105`, except it adds the `W:nnn` wait time.

- [ ] **`G28` homing**
  - In marlin, `G28` (no decimal) is homing, and the values are possibly provided without numbers, and the number (if provided) are ignored: `G28 X Y Z`
  - If no axes are provided, then it will home *all axes*.
  - As a side-effect, it [clears the auto bed-levelling rotation](https://github.com/MarlinFirmware/Marlin/blob/7bea5e5e5701de0b90b6c422c954337ce860bb0f/Marlin/Marlin_main.cpp#L3414), and sets the "active toolhead" to `T0`.
  - Will home Z first *if* Z homes up, then X, then Y. Otherwise it'll home X, then Y, then Z.

- [ ] **`G29` bed tramming**
  - This will probe three (internally stored) points, then activate bed levelling


## Unsupported

