# Marlin Compatibility Mode

### Why have a Marlin Compatibility mode?

For 3D printing there are a large number of utilities that are written to speak to [Marlin](https://github.com/MarlinFirmware/Marlin) or various other mostly-compatible variants. Even though there are several things that don't translate between Marlin and g2core, we can make an effort to at least accept gcode sliced for Marlin, and to be able to conditionally speak the Marlin protocol.

This makes g2core advanced 3d printing capabilities, such as instant feedhold and 6th order acceleration management  accessible to upstream tools that speak Marlin. It also gives the authors of those various utilities (assuming that they are still being maintained) a migration path to native g2core support should they wish to do so.

Marlin compatibility support is viewed as a transition option to ease migration, and may be declared "done" once it's achieved broad enough support.

### What It Is

The Marlin compatibility mode supports the most-used gcode/mcodes used in Marlin, and enough of the Marlin communications protocol to be able to drive a g2core 3d printer from a Marlin sender. Supported codes are listed on this page.

### What It's Not

The Marlin compatibility mode is not a full 100% work-alike for Marlin, although some additional commands may be added over time. You have to use the g2core protocol for these:

* Configuration of the machine and various parameters
* Setup of motors, heaters, etc.
* Tuning PID, limit switch polarity, etc.

## Concepts Cheat Sheet

There are a few concepts that convert more-or-less between g2core and Marlin, so they will be listed in this table.

| g2core | Marlin | Notes |
| :---: | :---: | --- |
| `{"r":...,"f":[...]}` | `ok` | In response to every sent gcode, g2core will return a response JSON, and Marlin will return `ok`. <br\> (Marlin has an ["advanced `ok`"](https://github.com/MarlinFirmware/Marlin/blob/7bea5e5e5701de0b90b6c422c954337ce860bb0f/Marlin/Marlin_main.cpp#L8704) option as well.) |
| `{"he1st":210}` | `M104 S201 T1` | Set temperature of extruder 1 to `210`. |
| `{"he1st":n}` | `M105` | Request the set temperature of extruder 1 (g2core) or current extruder and bed (Marlin). |
| `Nxxx` (gcode) | `Nxxx`&nbsp;(gcode)&nbsp;`*`&nbsp;(checksum) | [Marlin supports a checksum, and has special requirements on line numbers](http://reprap.org/wiki/G-code#Special_fields), where g2core accepts arbitrary line numbers and doesn't support a non-standard checksum. |
| `{"out4":0.5}` | `M106 S128` | Set output on the fan (or generic PWM output `4` for g2core) to 50%. |

## References

* [Official Marlin Repo](https://github.com/MarlinFirmware/Marlin)
* [Ultimaker2Marlin Repo](https://github.com/Ultimaker/Ultimaker2Marlin)
* [RepRap Gcode list](http://reprap.org/wiki/G-code)

## Marlin Serial Communications Protocol

### Status report (SR) analog

The Marlin communications protocol is fairly simple, but also somewhat lossy. It's expected that requests (such as `M105`) can be ignored and time out on the host side.

There's also no clear indication as to what's "part of the tape" gcode and what's a polling request such as `M105`.

In some circumstances, such as during a `M109` or `M190`, progress updates will be printed without polling roughly every second. Other than that and the case of [`M155`](http://reprap.org/wiki/G-code#M155:_Automatically_send_temperatures), all status updates must be polled for.

### Response (R) analog

See [Replies from the RepRap machine to the host computer](http://reprap.org/wiki/G-code#Replies_from_the_RepRap_machine_to_the_host_computer) for some documentation.

When any line is sent, an `ok` is returned as soon as it is optionally verified and parsed.

`ok` may be followed by temperature codes in the form `T:93.2 B:22.9` (generally in response to `M105`) or position codes in the form `X:9.2 Y:125.4 Z:3.7 E:1902.5 ...` (generally in response to [`M114`](http://reprap.org/wiki/G-code#M114:_Get_Current_Position)). See implementation for `M114` in [Marlin](https://github.com/MarlinFirmware/Marlin/blob/RC/Marlin/Marlin_main.cpp#L5948-L5965) and [Ultimaker2Marlin](https://github.com/Ultimaker/Ultimaker2Marlin/blob/1dfce5af3e9ab960078c1e6664b1e01a31609946/Marlin/Marlin_main.cpp#L1628-L1645), as they differ in output after the `E:`.  *(Q: Which extruder is E? Is E: in volumetric units for Ultimaker2Marlin?)*

If the line started with `Nxxx` and contains `*nnn` (before any comment, which begins with `;`), then the `xxx` and `nnn` are verified as the next sequential line number and checksum, respectively.

The checksum is the XOR of all characters up to but not including the `;` (Cura discards everything from `;` on), then the checksum is appended to the line (before any `;`) with the sprintf style string `"*%d"`. See the [Cura code](https://github.com/Ultimaker/Cura/blob/master/plugins/USBPrinting/USBPrinterOutputDevice.py#L557-L559) for an example:

```python
checksum = functools.reduce(lambda x,y: x^y, map(ord, "N%d%s" % (self._gcode_position, line)))
self._sendCommand("N%d%s*%d" % (self._gcode_position, line, checksum))
```

The line numbers (`Nxxx`) must be sequential, or Marlin will assume one was missed.

If verification fails, it'll respond with a `Resend: nnn` where `nnn` is the last seen line number + 1. The host is then expected to send that line again.

## Marlin Gcode Handling

Note that Marlin has several branches, and there are various "flavors" of gcode that work in various Marlins. This is partially controlled by compile-time options, but some variants may require using a completely different Marlin fork. (Needed: clear documentation of UltiGCode and Volumetric RepRap gcode.)

Gcode on Marlin must be all uppercase. g2core accepts both upper- and lower-case mixed.

On Marlin, motion of the extruder is a mixture of `E` and `T` words, where on g2core `A`, `B`, and `C` are currently used. (This is planned to change to using spindle-based controls.)

On g2core, `G0` does not accept an `F` word and `G1` requires one (but it is 'sticky,' so it only be included once, and from then on when it needs changed). In Marlin, G0 is [the same as G1](https://github.com/MarlinFirmware/Marlin/blob/RC/Marlin/Marlin_main.cpp#L2930-L2962).

## Marlin comment Handling

Generally, comments are handled the same way in Marlin and g2core.

In Ultimaker2Marlin there is some header gcode comments in the form [`;FLAVOR:UltiGCode`](https://github.com/Ultimaker/Ultimaker2Marlin/blob/1dfce5af3e9ab960078c1e6664b1e01a31609946/Marlin/UltiLCD2_menu_print.cpp#L393), etc. Here's an example:
```gcode
;FLAVOR:UltiGCode
;TIME:45010
;MATERIAL:40112
;MATERIAL2:0
;NOZZLE_DIAMETER:0.4
;Generated with Cura_SteamEngine 2.3.1-g2core-0.9

;LAYER_COUNT:240
;LAYER:0
M107
G10
G0 X60.763 Y31.538 Z0.27
;TYPE:SKIRT
```

Even when exporting "RepRap" flavor gcode, Cura prints some of these:
```gcode
;FLAVOR:RepRap
;TIME:16333
;Generated with Cura_SteamEngine 2.3.1-g2core-0.9
...
;LAYER_COUNT:120
;LAYER:0
```

## Supported `Gxxx` and `Mxxx` codes (different from normal g2core behavior)

- [ ] **`G0`**
  - On Marlin, `G0` is the same as `G1`.

- [ ] **`G28` - homing**
  - In Marlin, `G28` (no decimal) is homing, and the values are possibly provided without numbers, and the number (if provided) are ignored: `G28 X Y Z`
  - If no axes are provided, then it will home *all axes*.
  - As a side-effect, it [clears the auto bed-levelling rotation](https://github.com/MarlinFirmware/Marlin/blob/7bea5e5e5701de0b90b6c422c954337ce860bb0f/Marlin/Marlin_main.cpp#L3414), and sets the "active toolhead" to `T0`.
  - Will home Z first *if* Z homes up, then X, then Y. Otherwise it'll home X, then Y, then Z.

- [ ] **`G29` - bed tramming**
  - *Not suppored in Ultimaker2Marlin*
  - There are two types of `G29` - "mesh bed leveling" and "auto bed leveling." Each takes different parameters. *Here we'll discuss "auto bed leveling" with `AUTO_BED_LEVELING_3POINT` compiled in only.*
  - This will probe three points (internally stored and compiled-in as [`ABL_PROBE_PT_[123]_[XYZ]`](https://github.com/MarlinFirmware/Marlin/blob/182a1c949f27fd699d60cf971d88ecc3c8a7966d/Marlin/example_configurations/delta/generic/Configuration.h#L936-L941)), then activate bed tramming ("leveling").
  - The three points are stored in the firmware, and `G29` starts a "canned cycle" much like homing.

- [ ] **`G10`, `G11` - retract and unretract (prime) filament**
  - Retracts or primes filament according to settings of `M207`.

- [ ] **`M82`, `M83` - set extruder to absolute/relative mode**
  - `M82` sets the `E` to be absolute, while `M83` makes it relative.
  - *Not implemented in Ultimaker2Marlin, since E is volumetric and always effectively absolute and in volume.*

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
  - Cura [parses](https://github.com/Ultimaker/Cura/blob/master/plugins/USBPrinting/USBPrinterOutputDevice.py#L492-L501) looking for `/T: *([0-9.]*)/` and `/B: *([0-9.]*)/` only, and the rest appears to be ignored.

- [ ] **`M106`, `M107` - set fans speed/turn fan off**
  - `M106` turns the fan indicated with `Px` to the speed indicated with `Snnn`, or 255 by default.
    - `Snnn` has values for `nnn` of integers between 0 - 255, and is silently clipped to those values.
  - `M017` turns off the fan indicated in `Px`.

- [ ] **`M117` - display on LCD**
  - This is similar to the `MSG` mechanism in g2core, except it's intended for driving an LCD display.

## Unsupported
