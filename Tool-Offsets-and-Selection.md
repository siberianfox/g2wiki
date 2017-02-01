This page is based on, but not identical to, the following references: 

- [LinuxCNC Gcode reference](http://linuxcnc.org/docs/devel/html/gcode/g-code.html)
- [Tormach G43 Page](http://www.tormach.com/g43_g44_g49.html)

## Tool Selection and Tool Offsets
g2Core implements tool selection and tool changes using standard Gcodes and M codes. In addition, the tool table can be queried and set using [JSON commands](#json-commands) and current offsets and current tool can be queried using JSON.

The tool table can hold offsets for up to 32 tools, 1-32. Tool zero is not used. Use the Tn command to select a tool. Selecting a tool will not use that tool - it just stages it. To use the tool program M6. These can be performed on the same Gcode line. Examples:

`T12 M6`<br>
`M6 T12`  (order is unimportant)

Each tool can have offset values programmed in the tool table. Typically offsets are for the Z axis to compensate for different tool lengths or positions. This function is sometimes referred to as Cutter Length Compensation. However, offsets can be applied in any of the 6 dimensions XYZABC. This supports using tool offset for applications such as changing between multiple extruders on the same carriage. The term 'tool offset' is therefore used instead of 'tool length offset'.

To use a tool offset program: G43 Hn, where the H number is the desired tool number in the tool table. It is expected that all entries in this table will be positive. The H number may be zero, or the H word may be missing altogether; in either case an offset value of zero will be used. 

## Gcode Commands

	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	[T](#t-select-tool) | n | Select tool | `n` is tool number, 1 - 32
	[M6](#m6-change-tool) |  | Change to selected tool |
	[G10 L1](#g10-l1-set-tool-table-as-absolute-value) | Pn axes | Set tool offset as absolute value | `n` is tool number, 1 - 32
	[G10 L10](#g10-l10-set-tool-table-as-computed-value) | Pn axes | Set tool offset as computed value | `n` is tool number, 1 - 32
	[G43](#g43-set-tool-offset) | Hn | Set tool offset | `n` is tool number, 1 - 32
	[G43.2](#g43.2-set-additional-tool-offset) | Hn | Set additional tool offset | `n` is tool number, 1 - 23
	[G49](#g49-cancel-tool-offset) | | Cancel tool offset | 

### T Select Tool
Select the tool using Tn, where `n` is an index into the tool table, from 1 to 32. Selecting a tool out-of-range will result in a T_WORD_IS_INVALID error (181).

### M6 Change Tool
M6 moves the selected tool to be the tool in use. M6 does set the tool offset to the current tool G43 Pn is required (this is how Gcode works, so we did it this way, too).

_NOTE: At the current time M6 does not perform a manual tool change. Its only function is to make the selected tool the current tool._

### G10 L1 Set Tool Table as Absolute Value
G10 L1 sets the tool table for the P tool number to the values of the axis words. Axes that are not specified in the G10 L1 command will not be changed. 

It is an error if:
- The P number is unspecified
- The P number is not a valid tool number from the tool table
- The P number is 0

### G10 L10 Set Tool Table as Computed Value
G10 L10 changes the tool table entry for tool P so that if the tool offset is reloaded, with the machine in its current position and with the current G5x and G92 offsets active, the current coordinates for the given axes will become the given values. Axes that are not specified in the G10 L10 command will not be changed. This could be useful with a probe move as described in the G38 section.

It is an error if:
- The P number is unspecified
- The P number is not a valid tool number from the tool table
- The P number is 0

### G43 Set Tool Offset

Select a tool offset from the tool table indexed by the H word. H must be followed by an integer between 0 and N, where N is the maximum tool number. Omitting the H word has the same effect as H0.

This command causes no motion to occur. The next motion will take the new offset into account. Tool offsets are assumed to be positive numbers, but no restriction is placed on negative numbers. It is recommended that the tool offset be applied in the same line as the M6 Tn selection, but this is not required.

### G43.2 Set Additional Tool Offset

Applies a second, third or Nth offset to the current offset. This can be used to apply wear offsets or other summations. Otherwise this command obeys the behaviors of G43, above.

### G49 Cancel Tool Offset

Cancels the current tool offset.

## JSON Commands

### Read Tool Table
The tool table can be examined using the following commands:

	JSON | Command | Description
	-----|---------|-------------
	{tt1:n} | Tool table slot 1 | Returns tool table values
	{tt2:n} | Tool table slot 2 | As above
	... | ... | ...
	{tt32:n} | Tool table slot 32 | As above

These will return the tool table entry, as so:<br>
`{"r":{"tt1":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0}},"f":[1,0,8]}`

There is no command to read all tool table entries (no uber-group)

Tool table 0 is reserved, cannot be set, and is always all zeros

### Set Tool Table
It is also possible to set tool table values directly using JSON. We recommend using the standard Gcodes to set tool values during a job, and using JSON to pre-load the table or set values in advance of a job (job setup).

One or more values can be set in a JSON command. Values that are not specified are not changed. Examples

- `{"tt2":{"x":0.998,"y":0.998,"z":1.223,"a":0,"b":0,"c":0}}}` - Set all 6 axis values
- `{"tt2":{"x":0.998,"y":0.998,"z":1.223}}}` - Set XYZ only - ABC unaffected
- `{"tt2":{"x":0.998,"z":1.223}}}` - Set X and Z only - YABC unaffected

