g2core is used with third-party UI agents that cannot be expected to update on the same schedule as g2core. To avoid breaking extant copies of those UIs, interface changes must be done carefully.  Here is a list of do's and dont's to minimize breakage.

1. Do not delete functions.  When enhanced functionality is provided - for example the change in spindle control - the old functionality with the existing names and semantics should be retained. It is okay to implement the old names using the new internals, so long as the semantics are the same. The ideal approach is to gracefully extend the old scheme by adding new names that work in conjunction with the old ones, but when that is not feasible, add a complete coherent set of new names while retaining the old set.

2. Do not make the UI agent infer functionality from the version number.  Instead ensure that the presence of new functions can be discovered by probing for the new functions individually.

3. Make the probing process efficient.  Sending a command and waiting for a NAK (error message) is time-consuming if you have to probe for numerous functions.  Asking for a list of supported features is much better, since there is only one send-response delay.

4. Gracefully handle requests for unsupported features. In current versions, if the UI asks for a report format that contains unsupported fields, the g2core parser errors out and ignores fields after the one in error.  It would be better to continue parsing, accepting the valid fields, then issue a single report showing the rejected ones.

5. Do not change semantics of existing functions. A case in point is the desire to split the report format setting over multiple lines to work around line length limitations.  That is a good thing, but it should be done with a new name instead of by changing the behavior of the existing name.

I realize that the namespace can start to look cluttered with the proliferation of names, but that is a price that must be paid in an ecosystem of many independently-developed components.  The companies that have been most successful in the software industry are the ones who have done the best job of compatibility.

Line length limitations for json commands will start to be a problem.  I see two paths forward. The first is necessary for compatibility with existing usage, the second is a long term add-on.
1. Introduce some form of continuation line so that long json objects can be represented by multiple well-formed lines. This approach 
2. Migrate toward an improved transport protocol that is not so dependent on line semantics. My choice would be TCP, since it is perhaps the most battle-tested protocol on the planet, and has out-of-band messaging to handle jump-the-queue things like feedholds and override. It can be done efficiently over serial lines by using Van Jacobson header compression over SLIP. Over other channels like Ethernet and WiFi you just use it directly.  You just stream the gcode and let TCP handle the flow control.
