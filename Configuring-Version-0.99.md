**The configuration and settings documentation on this page is for firmware version 0.99.** FV 0.99 is currently in edge-100 branch undergoing testing. We are pretty comfortable with its functionality and stability, but have a few more functions to install and test before it's promoted to the edge branch. After a soak time in edge it will be promoted to the master branch.

The version number can be found as the fv variable in the startup JSON message, or by typing $fv. Version 0.99 encompasses builds 100.00 and later.

If you have an older firmware version:
* [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)

## Configuration Pages

- [Configuring Motors](Configuring-0.99-Motors)
  - [Power Management](Power-Management)
- [Configuring Axes](Configuring-0.99-Axes)
- [Configuring System Groups](Configuring-0.99-System-Groups)
- [Configuring General Purpose IO](Configuring-0.99-GPIO)
- [Configuring Other Groups](Configuring-0.99-Other-Groups)
- [Configuring Actions and Reports](Configuring-0.99-Actions-and-Reports)
- [Configuring 3D Printing Extensions](Configuring-0.99-3D-Printing-Extensions)

##Conventions Used for Config Documentation
- Examples show relaxed JSON input (e.g. `{fv:n}`). Strict JSON is also accepted in all cases (e.g. `{"fv":null}`)
- CMD means some command - aka the "name" of the name/value pair. CMDs are case insensitive.
- Underscore "_" means some numeric value
- "abcd" means some string value
- Motor examples use Motor 1, but any motor active for your platform is OK
- Axis examples use X axis, but any axis is OK unless otherwise noted
- All examples are in millimeter units

**A note about units**
- All data entry and display occurs in the prevailing units set in the Gcode mode. Set G20 for inches, G21 for mm. Query {unit:n} or $unit to find out where you are (G20=0, G21=1)
- Units entered in one system are correctly preserved and do not need to be re-entered or adjusted if the Gcode units mode changes.
- The Gcode default units setting `{gun:_}` sets the units the system "comes up in" during power-on or reset. So changing GUN will not change your units until you reset. Note: PROGRAM END (M2, M30) does not change the units. 

### JSON Cheat Sheet
All configuration commands are available using [JSON mode](JSON-Operation), which is the preferred access method if you are writing a UI or controller. Most commands are also available in [Text Mode](Text-Mode). The few commands that are available from only one or the other are noted, as are any commands the behave differently depending on the mode. The rough equivalence is:

| JSON | Text | Description |
| :--: | :--: | :-- |
| `{"cmd":null}` | `$cmd`   | Read value for command "CMD" in strict JSON mode |
| `{cmd:n}` | `$cmd` | Read value in relaxed JSON mode |
| `{"cmd":123.4}` | `$cmd=123.4` | Set value to 123.4 in strict JSON mode |
| `{cmd:123.4}` | `$cmd=123.4` | Set value in relaxed JSON mode |
| `{xvm:50000}` | `$xvm=50000` | Set X max velocity to 50000 as an example |

You can also use JSON to read an entire object - examples for motor and axis:
<pre>
{1:n}
{"r":{"1":{"ma":0,"sa":1.8,"tr":36.576,"mi":8,"su":43.74453,"po":0,"pm":0,"pl":0.45}},"f":[1,0,6]}

{x:n}
{"r":{"x":{"am":1,"vm":40000,"fr":40000,"tn":0,"tm":420,"jm":5000,"jh":20000,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":4,"zb":2}},"f":[1,0,6]}
</pre>

A command in JSON mode returns a response like the one below. The {} object contains the result, which in this case is a blank JSON object because we sent a Gcode command `G0 X10` and have set JSON verbosity to not report back in this case (`{jv:3}`, for example): 
```
{"r":{},"f":[1,0,7]}
```
The footer is an array of 3 elements:
- (1) Footer revision
- (2) [Status code](Status-Codes) - Short version: status=0 is OK, everything else is an exception.
- (3) RX buffer info (explained later)
- Note: The checksum that used to be in the footer has been removed as no-one was using it.
