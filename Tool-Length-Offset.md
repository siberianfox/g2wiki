References:
http://www.tormach.com/g43_g44_g49.html


Details:
To use a tool length offset, program: G43 H~, where the H number is the desired index in the tool table. It is expected that all entries in this table will be positive. The H number should be, but does not have to be, the same as the slot number of the tool currently in the spindle. The H number may be zero; an offset value of zero will be used. 


	Gcode | Parameters | Command | Description
	------|------------|---------|-------------
	[G43](#G43-Set-Tool-Length-Offset) | Hn | Set tool length offset | 
	[G43.2](#G43.2-Set-Additional-Tool-Length-Offset) | Hn | Set additional tool length offset | 
	[G49](#G49-Cancel-Tool-Length-Offset) | --none-- | Cancel tool length offset | 

### G43 Set Tool Length Offset

Select a tool offset from the tool table indexed by the H word. H must be followed by an integer between 0 and N, where N is the maximum tool number (currently 16). Omitting the H word has the same effect as H0.

This command causes no motion to occur. The next motion will take the new offset into account. Tool offsets are assumed to be positive numbers, but no restriction is placed on negative numbers. It is recommended that the tool offset be applied in the same line as the M6 Tn selection, but this is not required.

### G43.2 Set Additional Tool Length Offset

Applies a second, third or Nth offset to the current offset. This can be used to apply wear offsets or other summations. Otherwise this command obeys the behaviors of G43, above.

### G49 Cancel Tool Length Offset

Cancels the current tool offset.

## JSON Commands

### Read Tool Table
The tool table can be examined using the following commands:

	JSON | Command | Description
	-----|---------|-------------
	{tt1:n} | Tool table slot 1 | Returns tool table values
	{tt2:n} | Tool table slot 2 | As above
	... | ... | ...
	{tt16:n} | Tool table slot 2 | As above

These will return the tool table entry, as so:<br>
`{"r":{"tt1":{"x":0,"y":0,"z":0,"a":0,"b":0,"c":0}},"f":[1,0,8]}`

### Set Tool Table
It is also possible to set tool table values directly using JSON. We recommend using the standard Gcodes to set tool values during a job, and using JSON to pre-load the table or set values in advance of a job (job setup).

One or more values can be set in a JSON command. Values that are not specified are not changed. Examples

- {"tt2":{"x":0.998,"y":0.998,"z":1.223,"a":0,"b":0,"c":0}}}` - Sets all 6 axis values as indicated
- {"tt2":{"x":0.998,"y":0.998,"z":1.223}}}` - Sets XYZ only - ABC unaffected
- {"tt2":{"x":0.998,"z":1.223}}}` - Sets X and Z only - YABC unaffected

