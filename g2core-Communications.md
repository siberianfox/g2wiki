_This page describes the g2core communications options and protocol. It is intended for GUI developers and other low-level access. This page is useful if for some reason you need to write your own communications handler, or just want to know how it works._

**We strongly recommend using the [node-g2core-api](https://github.com/synthetos/node-g2core-api) nodeJS module, which already handles these communications details.**
**If you are building your own, we strongly recommend you use the linemode protocol described below.**

## Overview of Communications Model

A **host** is any computer that talks to a g2core board. Typically this is an OSX, Linux, or Windows laptop or desktop computer communicating over USB.

A **microhost** is a tiny linux system like a Beaglebone, Edison, Raspberry Pi, or NextThingCo CHIP. These usually talk to g2core over a direct serial UART - although they may also communicate over USB.

### Control and Data Channels

A **command** is a single line of text sent from the host to the board. A command can be either **data** or **control**.

* **Data**: Gcode lines are always data commands. Generally, unless a line is identified as a control, it's considered data.
* **Control**: JSON commands and single-character commands (`!` feedhold, `%` queue flush, `~` resume, etc) are considered as control commands. Note: JSON in active comments are actually data commands, as they are gcode lines.

The protocol distinguishes between data and controls, and always executes controls first.

## Line Mode Protocol
As of the g2core 100 builds we recommended using Line mode protocol for host-to-board communications. In line mode the host sends a few command lines to prime the board's receive queue, then sends a new line each time it receives a response from a processed line. g2core 100 builds also support character mode (byte streaming) which is deprecated and may be removed at a later time.

Line mode protocol is designed to prevent the serial buffer from either filling completely (preventing time-critical commands from getting through) while keeping the serial buffer full enough in order to prevent degradation to motion quality due to the motion commands not making it to the machine in a timely manner.

Data and Control commands are sent over the same link, and are sent to separate internal queues, allowing control commands to always take priority over data commands like Gcode. 

The protocol is very simple - "blast" 4 commands (text lines) to the board without waiting for responses. From that point on send a single line for every `{r:...}` response received. Every command will return one and only one response. **The exceptions are single character commands - which do not consume line buffers and do not generate responses.** 

* Recognized single character commands are:
  * `!` - Feedhold request
  * `~` - Feedhold exit (restart)
  * `%` - Queue flush
  * `^d` - Kill job (ASCII 0x04)
  * `^x` - Reset board (ASCII 0x18)
  * `ENQ` - Enquire communications status (ASCII 0x05) 

### Line Mode Reference Implementation
Here's a pseudocode reference implementation that's actually rather simple:

1. Prepare or start reading the list of _data_ lines to send to the g2core. We'll call this list `line_queue`.
2. Set `lines_to_send` to `4`.
  * `4` has been determined to be a good starting point. This is subject to tuning, and might be adjusted based on your results.
3. Send the first `lines_to_send` lines of `line_queue` to the g2core, decrementing `lines_to_send` by one for _each line sent_.
  * If you need to read more lines into `line_queue`, do so as soon as `lines_to_send` is zero.
  * If `line_queue` is being filled by dynamically generated commands then you can send up to `lines_to_send` lines immedately.
  * Don't forget to decrement `lines_to_send` for _each line sent_! And don't send more than `lines_to_send` lines!
4. When a `{r:...}` response comes back from the g2core, add one to `lines_to_send`.
  * It's **vital** that when any `{r:...}` comes back that `lines_to_send` is incremented. If one is lost or ignored then the system will get out of sync and sending will stall.
5. Loop back to 3. (Better yet, have 3 and 4 each loop in their own thread or context.)

Notes:
* Steps 3 and 4 are best to run in their own threads or context. (In node we have each in event handlers, for example.) If that's not practical, it's vital that when a `{r:...}` comes in that `lines_to_send` is incremented and that lines can be sent as quickly after that as possible.
* It is possible (and common) to get two or more `{r:...}` responses before you send another line. This is why it's vital to keep track of `lines_to_send`.
* Note that only _data_ (gcode) lines go into `line_queue`! For configuration JSON or single-line commands, they are sent immediately.
  * It's important to maintain `lines_to_send` even when sending past the `line_queue`.
    * Single-character commands will *not* generate a `{r:...}` response (they may generate other output, however), so there's nothing to do (see following notes).
    * JSON commands (`{` being the first character on the line) **will** have a `{r:...}` response, so when you send one of those past the queue you should still subtract one from `lines_to_send`, or `lines_to_send` will get out of sync and the sender will eventually stall waiting for responses. This is the *only* case where `lines_to_send` may go negative.
  * Note that `{"gc":"...}` Gcode line are considered control lines and bypasses motion planner buffer tests and are acknowledged immediately. Consider sending GCode as unwrapped text, _while observing linemode protocol_. See [issue 287](https://github.com/synthetos/g2/issues/287) for more detail.
  * Note that control commands, like data commands, must start at the beginning of a line, so you should always send whole lines. IOW, don't interrupt a line being sent from the `line_queue` to send a JSON command or feedhold `!`.
* **All** communications to the g2core **must** go through this protocol. It's not acceptable to occasionally send a JSON command or gcode line directly past this protocol, or `lines_to_send` will get out of sync and the sender will eventually stall waiting for responses.

## More Details

### ENQ/ACK - Checking for Clean Startup
USB does not always connect reliably. Sometimes you have to hit it a few times (Windows is the worst). The ENQ/ACK dialog is provided as a test of connectivity and board health. The protocol is:

* Send ENQ - send a single-character ENC (0x05)
* Returns `{"ack":true}\n` - Indicating that the serial communication and board's main loop are operational

Note that if Marlin Compatibility Mode is enabled board startup is delayed by 2 seconds from power-on. This is to allow for Marlin's STK-500 dialog to complete. If Marlin Compatibility Mode is not enabled board startup occurs within a few milliseconds of power-on. 

### Order of Operation

To somewhat repeat what was just said, there are three distinct places that a line may be executed:

1. Command is executed when it's read by the serial input handler - Control commands such as JSON (first character of the line is a `{`) and single-character commands (`!`, `%`, etc) fit in this category.

    * Single-character lines must be the first character on a line, and do not need a following line-ending `\n` *but can have one.* Some serial-port libraries will auto-flush on a line-ending, making it better to add a `\n` to the end of a `!`.

    * Multiple single character commands may be put together, such as `!%` with nothing in between. If there is a space, such as `! %` then the `%` **will not be seen as a queue flush**. (This is to deal with Inkscape that uses % as a comment character in CAM output, so there's some stateful dancing that needs to be done to keep this from being a problem).

2. Command is executed immediately when it's read from the serial input queue - Data commands that are not synchronized with motion fit in this category. `M100.1` is a command that has this behavior. See `M100.1` in the example below.

3. Command is executed synchronized with motion (i.e. executed from the planner queue). Everything that's not part of one of the above categories, including most gcode, fits into this category.

For example, if the internal buffer contains this:
```gcode
G54
G0 X20
M100 ({out1:0.5})
M100.1 ({g55z:-0.1})
G0 X10
G55
G0 X0
{sr:n}
```

They will be executed in this order:

1. `{sr:n}` - JSON is executed as soon as it's seen in the buffer, past anything else.

    * Note that it has to be in the g2core buffer before it can be seen. If the host has commands buffered, it might not be seen immediately.

2. `G54` - this command is synchronized with motion, so it's queue to the planner. Assuming the planner is empty it is executed shortly thereafter. 

3. `G0 X20` - the move will be queued to the planner and start executing almost immediately

4. `M100 ({out1:0.5})` - the JSON command `{out1:0.5}` will be *queued* to the planner to execute after the move is completed.

5. `M100.1 ({g55z:-0.1})` - will be read from the serial queue after the preceding M100 command is queued to the planner, and the JSON command `{g55z:-0.1}` will be executed *immediately*. The `G55` offsets will thus be adjusted, but the Select G55 Coordinate System command itself has not yet been executed, so the machine is still in G54 and reflecting these offsets.

6. `G0 X10` - another move will be queued

7. `G55` - the `G55` offsets will be queued to activate for further moves.

8. `G0 X0` - this move will be queued to execute in the `G55` coordinate system.

Note that the `{out1:0.5}` will have only been queued to happen, and will only actually be executed in sync with the motion, while the machine is at `X20`.


**Note:** g2core 100 builds can enumerate as dual-endpoint USB (two serial ports show up when connected). Line mode works when connected to a single port or to both ports.

* When connected to both ports, the first port opened is the control channel, and the second port is considered data. Only raw JSON and single-character commands should be sent over control channel, and everything else (including gcode) should be sent over the data line. All responses from g2core will be sent over the control channel.
* When connected to a single port, all data must go through the same channel, and this is essentially the same as if you are communicating over a raw UART channel.
* _Dual-endpoint USB has been deprecated, and will be made a custom compile-time setting at some point._ In the future, only a single serial-port will enumerate. When building a UI, it's recommended to support using a single port if the second one is not available.

## Problem Description - the "Bucket Brigade"

[![Bucket Brigade](http://www.stepneyct.org/history/ht/images/stop_17e_430x195.jpg)](http://www.stepneyct.org/history/ht/stop17.html)

With a single channel of communication, either UART or USB, we have two single-file lines of bytes: one going to the g2core, and one coming from the g2core.

In both cases, there are often several "players" involved between the two endpoints of the channel. For example, for USB, there is:

  * Your UI application →
  * The language's serial buffers →
  * The OS serial USB driver buffers →
  * The OS USB buffers →
  * The g2core USB peripheral buffers →
  * The g2core serial buffers, then, finally →
  * The g2core line parser.

This amounts to a "bucket brigade," where once you pass the line into the serial system, it has to pass through several "hidden" layers (there may be more or less than listed above) before it gets to the g2core, and then there's a buffer before it can be parsed and handled.

This is generally fine, as long as you don't need to stop the flow of commands. For example, to execute a feed-hold you send a `!` character at the beginning of a new line. It has to get through all of those buffers before it is even seen by the g2core. So, if there are too many lines in the buffer before the g2core, the feedhold may take a noticeable (and unacceptable) amount of time to go into effect.

To complicate matters, the g2core cannot blindly read lines, but only reads lines as there is room for the parsed results. This generally means that as a line is read and parsed, room is made in the internal serial buffer. If the motion buffer is full, however, there is no room for new moves to be stored, so g2core will stop reading and parsing _data_ lines until room is made in the motion buffer. It will continue to read _control_ lines in any case.

To help with this the g2core serial system will "pre-parse" the incoming serial buffer, even when data lines are no longer being read (due to the motion buffer being full, a feedhold in place, etc), and look for lines that begin with "{" or one of the single-character commands (`!`, `~`, `%`, `^D`, `^X`, etc.). If it locates one of these lines, it will mark it as a control line and the next control-line-read will see it and execute it immediately.

So, as long as the serial buffer is not completely filled with data lines, there is at least room for a single-line command and generally room for a JSON command as well.

There are other issues that are dealt with in other ways, such as if the serial buffer _does_ fill completely (even if very briefly) we need to ensure that we don't lose data. This is mostly a concern with UART, and in that case we require some form of _flow control_. Currently we only support hardware flow-control, using the additional `RTS`/`CTS` lines. Linemode is _not_ a replacement for flow-control, and will not prevent lost data due to buffer overflow on it's own.

One further note: Due to the way g2core plans motion in real-time, if the gcode commands request moves of very brief duration, and the serial buffer isn't kept full enough of moves, then there will be a noticeable degradation in velocity and in some configurations the machine will exhibit a noticeable "stutter" as it executes each move on it's own or in small groups.


