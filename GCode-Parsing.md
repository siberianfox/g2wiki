:toc: macro
:toclevels: 4
:icons: font

toc::[]

// See http://asciidoctor.org/docs/user-manual/#basic-document-anatomy for ASCIIDOC documentation

# G Code Parsing Notes

## Valid G Code format

### References
- https://www.nist.gov/customcf/get_pdf.cfm?pub_id=823374[NIST specification]
- http://linuxcnc.org/docs/html/gcode/overview.html#_g_code_overview[LinuxCNC G Code documentation]
- http://www.tormach.com/machine_codes_gcodes.html[Tormach G Code documentation]
+
NOTE: Tormach now uses LinuxCNC.


Overview::
- G Code is a line-based static file format. It is first split on line endings (`\n`, `\r`, or `\r\n`), then each line is processed in turn.
- Each line breaks down into an optional "line number", zero or more "words", and zero or more "comments."
- There are rules on what words can be on a line together, and what words are required with other words.
- There are also "modal" words, which become implicit on later lines until they are replaced by another word of the same "modal group."

### Line numbers

`N...` where `...` is a number from 0 - 99,999. This looks like a "word" but actually is not. It is optional but must be at the beginning of a line if at all.

### Words

Other than line numbers, a character followed by a number is considered a "word".

Example words::
- `G0`
- `x15`

Words can be rearranged as needed, and have the line will not change the meaning.

NOTE: There are already rules in place for what words are allowed on the same line. We are to fail the line (and the job?) if this is violated.

### Modal Words and Modal Groups

NOTE: This table is directly copied from Table 4 of page 20 the <<References,NIST>> specification.

.Modal Groups
|===
h|The modal groups for G codes are:

|group 1 = {`G0`, `G1`, `G2`, `G3`, `G38.2`, `G80`, `G81`, `G82`, `G83`, `G84`, `G85`, `G86`, `G87`, `G88`, `G89`} motion
|group 2 = {`G17`, `G18`, `G19`} plane selection
|group 3 = {`G90,` `G91`} distance mode
|group 5 = {`G93`, `G94`} feed rate mode
|group 6 = {`G20`, `G21`} units
|group 7 = {`G40`, `G41`, `G42`} cutter radius compensation
|group 8 = {`G43`, `G49`} tool length offset
|group 10 = {`G98`, `G99`} return mode in canned cycles
|group 12 = {`G54`, `G55`, `G56`, `G57`, `G58`, `G59`, `G59.1`, `G59.2`, `G59.3`} coordinate system selection
|group 13 = {`G61`, `G61.1`, `G64`} path control mode

h|The modal groups for M codes are:

|group 4 = {`M0`, `M1`, `M2`, `M30`, `M60`} stopping
|group 6 = {`M6`} tool change
|group 7 = {`M3`, `M4`, `M5`} spindle turning
|group 8 = {`M7`, `M8`, `M9`} coolant (special case: M7 and M8 may be active at the same time)
|group 9 = {`M48`, `M49`} enable/disable feed and speed override switches

h|In addition to the above modal groups, there is a group for non-modal G codes:

|group 0 = {`G4`, `G10`, `G28`, `G30`, `G53`, `G92`, `G92.1`, `G92.2`, `G92.3`}

|===

### Comments

Anything between `(` and `)` are considered a comment. Any line can have multiple comments on the same line. If the close parenthesis is missing, it is supposed to be an error.

NOTE: We might ignore this case of the missing `)`, but perhaps we shouldn't. If there is a missing `)` then it may indicate the line is malformed and won't be interpreted int he way it was intended. The `;` and `\` characters both provide alternate mechanisms to end the line early if that was the intent.

Comments are **not** to be nested. In other words, the charater `(` is not allowed after a `(` and before a `)`.

Any character after a `;` is considered a comment. This type of comment is **not** to be interpreted, but simply thrown away. We can effectively treat a `;` as a `NULL` character and the end of the line.

*Addition:* Some malformed G Code generators wrongly use a `%` character as an alternative to `;`. We will support that through complex heuristics to seperate it from our other uses of the `%` character. ^Citation_Needed^

Though not technically a "comment," if the first character is a `/` then the line is _optionally_ to be skipped if the *block delete switch* is on.

### Active comments

We are adding the concept of the "active comment" on top of the NIST comments. http://linuxcnc.org/docs/html/gcode/overview.html#sec:comments[LinuxCNC] has a similar concept (and is the origination of the term "active comments"), but we are intentionally limiting them to comment-wrapped-JSON.

For _our_ active comments, we are limiting the active comments to JSON immediately wrapped in parenthesis (starting with `({` and ending with `})` ). Not space is allowed between the `(` `)` and the `{` `}` characters.

.Examples
[source,gcode]
----
({ wait:{ io1:t }}) <1>
( {wait:null} ) <2>
----
<1> Active comment. There are no spaces between beginning `(` and `{` or between `}` and `)`.
<2> Normal commment, due to the space between the `(` and `{`.

Following the rules of comments (active or not), we can have multiple comments. We can also have words before, after, and between comments. This means we can have multiple active comments as well.

## Normalization

In order to simplify the parsing and processing of the G Code line, we want to "normalize" it as a string of bytes into a simpler form. The trick of this is that we can't accidentally change the "meaning" of the line in the process.

### Steps of normalization

NOTE: It's assumed that several of these steps may be done in parallel. I list them individually for clarity.

. Replace `;` and `%` when found with a `NULL` and ignore the rest of the string.
. Strip out `(` and everything up the the first following `)` _only_ when the first character after `(` is not `{` *or* `M` followed by `S` followed by `G`.
. Since there can be multiple `MSG` comments, and we only want one. Throw away the rest.
.. _(Option 1)_ Count MSG comments in one pass, and strip all but the last one in a second pass. (This honors NIST's statement that only the last one is to be kept.)
.. _(Option 2)_ Ignoring the NIST spec, once we have seen one `MSG` comment, we delete the rest as normal comments.
. Since there can be multiple active comments, and position relative to the G Code words doesn't matter, we can merge them.
.. Move all active comments to the end of the line (buffer?) and all G Code words to the beginning.
+
NOTE: At this point we will be parsing the JSON in-line. Parts of this string may get copied for use in a command buffer. We don't need to worry about that here, though.
.. Merge all active comments on a line into one JSON object, replacing the `)}{(` with a `,`.
. Capitalize all of the G Code words.
. Remove extraneous zeros from numbers that would cause them to be treated as octal.

### Example normalization

[source,yaml]
----
# Incoming raw line
N1 g0 (blah)({wait:{i01:n}}) x001(MSG Say this)({camera:1}) ; say something and wait

# First pass, strip extra bits
N1g0({wait:{i01:n}})x001(MSG Say this)({camera:1})

# Rearrange so words are at the front and comments we're keeping at the end.
# We also merge JSON comments
N1g0x001(MSG Say this)({wait:{i01:n},camera:1})

# Capitalize and strip leading zeros from the G Code words
N1G0X1(MSG Say this)({wait:{i01:n},camera:1})
# ^- GCode ^- MSG    ^- JSON
# Now we are done. Not we maintained pointers to:
# The beginning of the G Code
# The MSG comment
# The active comment JSON
----
