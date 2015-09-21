_This page contains topics relevant to development - exclusive of the toolchains. See the dedicated pages for setting up projects on different platforms for that information_

###Main Loop and Continuations - Adding New Tasks
This topic discusses how to add tasks to the main loop.

The controller_run() / _controller_HSM loop in controller.cpp is the main "throttle" that regulates how fast the system can operate - i.e. how fast does the code run through the loop, and therefore at what rate the system can ingest new Gcode blocks. So you don't want to inject any latency that can be avoided. 

Read the header notes for controller_run() in controller.cpp first.

New tasks can be added as callbacks inserted into the main loop. Any task that is added to the main loop must behave properly. That is: 
- Return as fast as you can (preferably STAT_NOOP) if you have nothing to do
- Return STAT_OK if that task's end state would not block subsequent tasks from executing
- Return STAT_EAGAIN if that task's end state should prevent lower priority tasks from executing. Eagain will force the main loop to start over from the top.
- A callback should generally not take more than +/- 2 milliseconds to execute (ideally less), and must not block internally. If it encounters a blocking condition it should return control to the main loop, possibly with STAT_EAGAIN if lower-priority tasks are dependent on its completion.

Write tasks as co-routines with an entry point to initiate the task, and a continuation installed as a callback to continue the task. A singleton structure can be used to keep state between initiation and repeated calls to the continuation. 

See plan_arc.cpp, cm_arc_feed() and cm_arc_callback() for a good example of an entry function and a continuation. See st_motor_power_callback() for an example that uses the system timer to schedule events.
