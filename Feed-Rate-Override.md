**WARNING: This function is experimental and not complete. Expect changes and refinements to the functionality an the interface functions. Check back for updates**<br>
PAGE STARTED: 18-Aug-15 - ash

##Manual Feed Rate Overrides

###Prerequisites
- Requires g2 edge (089.02) or later

###USB Setup and Connection
It's best to open g2 in USB dual-endpoint mode. 
- On the mac you will see something like `usbmodemfd1311` and `usbmodemfd1313` in the device list.
- Open one of these first. This port becomes a Control and Data port.
- Open the other port. This becomes the Data port and the first becomes the Control port.
  - Send Gcode to the data port
  - Send everything else to the control port
  - All responses flow back to the host over the control port. You do not need to monitor the data port

###Feed Rate Override Usage
Turn on the enables before executing Gcode. Adjust the MFO factor during motion.
- Set master override enable to ON `{m48e:1}`
- Set manual feed rate override ON `{mfoe:1}`
- Enter / adjust manual feed rate override factor `{mfo:N}` where 0.05 < N < 2.00

###Known Limitations
- Does not affect moves already in the planner. Only affects incoming moves.
- Does not currently ramp between velocities - relies on jerk to limit rate-of-change.

###Bugs and Comments
- Please report and bugs or other comments to g2 Issues