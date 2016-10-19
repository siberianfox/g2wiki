Character | Command | AKA | Description
----------|---------|-----|-------------
! | Feedhold | Pause | Stop movement immediately. Maintains positional accuracy
~ | Cycle Start | Resume | Resume movement from stopping point
% | Queue Flush | | Empty planner buffer. Used for jogging and other cycles

To pause motion send a feedhold character (`!`) character during motion. Movement will decelerate to a stop while maintaining positional accuracy. To resume motion send a cycle start tilde character (`~`).

If you wish to not perform the remainder of the move after the feedhold send a queue flush character (`%`) after the feedhold character. Motion will start with the next Gcode command entered. A cycle start is not necessary after a queue flush.

## Jogging Using Feedhold and Queue Flush
To jog using feedhold and queue flush do the following:
* Start the jog by issuing a move in the jogging direction at the velocity you want for the jog
* Send a ! when the user wants the jog to stop
* Send a % to flush the remainder of the move. These should each beon their own line, but can be sent together in a single transmission: `"!\n%\n"`

Nudge moves to fine-tune the jog can be performed by issuing short Gcode moves such as `G91 G1 F100 X0.001`

Remember to set the system back to absolute mode by issuing a `G90`.
