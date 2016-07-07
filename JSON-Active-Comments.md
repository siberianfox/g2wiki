This page describes using Gcode comments to carry JSON commands such that they can extend existing Gcode functions or insert control functions that execute synchronously with the Gcode.

##Gcode Comments
Gcode comments in G2 possess a number of features that extend their capabilities beyond classic Gcode. Classic Gcode only defines parentheses comments `(...)` and message comments `(msg...)`. Additionally, a slash character `/` in the first character is a block delete, which can be considered a type of comment. 

G2 handles comments as so:

- Classic Gcode comments that start with a `(` and end with a `)` are known as "inline comments" and will be removed, UNLESS they are an active comment

- Gcode Message comments are active comments with the letters `msg` (case insensitive) immediately following the open parenthesis: `(Msg....)`
  - The string enclosed by the trailing `g` and the closing paren will be returned in the next status report as a `"msg":"...."` tag
  - If there is a space or anything else between `(` and the `m` then it will be treated as a normal comment
  - Only one message comment is allowed per Gcode block

- JSON comments are active comments with JSON immediately inside the parentheses: `({...})`
  - JSON comments carry content that GCode can't express, but is intrinsically part of the command in that Gcode block
  - If there's a space or anything else between the `(` and the `{` then it will be treated as a normal comment

- Multiple inline comments `(...)` are allowed, and active comments (message and JSON) in between will be retained

- Multiple active comments are allowed, and will be combined and treated as if they were separated by commas
  - Ex: `M100 ({he1st:200}) ({he2st:210})` will be treated the same as `M100 ({he1st:200, he2st:210})`
  - Message comments are converted internally to JSON comments, so these may also be mixed with JSON comments

- Comments that start with a semicolon `;` end the line -- everything including and after the `;` will be ignored, including any active comments. Some Gcode generators use semicolons as comments

- Comments that start with a percent sign `%` (as some Inkscape Gcode generators do) are converted to semicolons and end the line -- everything including and after the `%` will be ignored. A percent sign received during a feedhold is treated as a queue flush

- Lines that start with a forward slash `/` in the first character are block delete lines. These lines are omitted in their entirety

### Valid Comment Cases

| Valid comment cases       | Notes: |
| --- | --- |
| `G0X10`                      | command only - no comment |
| `G0X10 (comment text)`       | command with comment |
| `G0X10 (comment text`        | acceptable, but a warning may be issued for the comment not being terminated |
| `G0X10 ;comment text`        | comment delimited by semicolon |
| `(MSGSend this string)`      | will report back as `"msg":"Send this string"` |
| `(comment text)`             | there is no command on this line |
| `G0 (traverse) X10 (to X ten) Y12 (and Y twelve)` | Command `G0X10Y12` with multiple inline comments |
| `M100 (set heater temp to 210:) ({he1st:210})` | Command with inline comment and active comment |
