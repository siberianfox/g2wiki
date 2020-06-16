_This page contains an overview of motion processing. It was originally written as a response to a question of how to separate the step/direction outputs from the rest of the planner._

You can break the step-direction motion out from the planner, as they are independent (but currently deeply symbiotic) systems. The challenge lies mostly in the timing.

There are six  stages to get from gcode to steps:

1. parser, pretty obvious, but it also puts moves into the planner as blocks

2. back-planner, where it takes the most recently inserted block (and what would be the last to execute) and plans from there “backward” to the currently executing block. This stage only handles feedrate limits to ensure the machine can stop before it runs out of moves, and setting the cornering speeds.

3. just-in-time (JIT) forward-planner, where it computes the head (jerk-bound acceleration), body (cruise), and tail (deceleration) sections of the next block past the one currently running

All of these parts so far we generally lump together and call the “planner” conversationally.

4. Execution, or “the exec,” where it breaks the head, body, and tail sections into the next linear-acceleration segment that is generally between 0.25ms to 1ms in length (depending on hardware and configuration), and computes the start and end velocity, number of steps for each motor (Via kinematics), and duration of said segment and puts it where the loader can get it

5. The loader, which takes the next segment and loads it into the runtime between steps

6. The runtime, which is the part that generates the steps of the currently loaded segment. The runtime never stops. Even when motion is stopped, the runtime is constantly running.

As for the timing I mentioned before, our system is designed to be pull, where:
- the runtime informs the loader when to run,
- which then signals the exec to prepare a new segment,
- which tells the JIT to plan the next sections of the next block,
- and when the exec is done with a block and ejects it, it tells the parser there’s a space to parse a new block,
- which tells the XIO (serial port, usb, etc) to read a new line of gcode

So, if you remove the runtime and exec and move them to a new processor, you’ll have to handle the timing and pull of those subsystems another way.

Timing is also important in that some segments (as brief as 0.25 ms) are a whole block which was a gcode segment. UART and USB serial (generally on the host side) is not that fast, generally, and so we have to throttle those by other means.

Basically if you take all of stepper.cpp and the last third of plan_exec.cpp and move them to another processor, then you’ll have achieved your goal (of separating out the step generation from the motion planning). Of course you have to make them talk and sync up as I mentioned before.
