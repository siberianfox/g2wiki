**WARNING: This function is experimental and not complete. Expect changes and refinements to the functionality an the interface functions. Check back for updates**<br>
PAGE STARTED: 18-Aug-15 - ash

## Manual Feed Rate Overrides

### Prerequisites
- Requires g2 edge (089.02) or later

### USB Setup and Connection
It's best to open g2 in USB dual-endpoint mode.
- On the mac you will see something like `usbmodemfd1311` and `usbmodemfd1313` in the device list.
- Open one of these first. This port becomes a Control and Data port.
- Open the other port. This becomes the Data port and the first becomes the Control port.
  - Send Gcode to the data port
  - Send everything else to the control port
  - All responses flow back to the host over the control port. You do not need to monitor the data port

### Feed Rate Override Usage
Turn on the enables before executing Gcode. Adjust the MFO factor during motion.
- Set master override enable to ON `{m48e:1}`
- Set manual feed rate override ON `{mfoe:1}`
- Enter / adjust manual feed rate override factor `{mfo:N}` where 0.05 < N < 2.00

### Known Limitations
- Does not affect moves already in the planner. Only affects incoming moves.
- Does not currently ramp between velocities - relies on jerk to limit rate-of-change.

### Bugs and Comments
- Please report and bugs or other comments to g2 Issues

## Implementation notes

- Override adjustments are not to be applied all at once, as this may not give the operator time to react to too-aggressive of a speedup during tooling.
  - In order to provide time for the operator to react to the changes, a "ramping" function is to be used to apply the requested override over time. A linear ramping from one override setting to the latest requested setting will be used.
- At no point will an override allow any of the configured limitations to be violated. The override is *only* to adjust the incoming requested feedrate (and separately, the traverse rates). None of the jerk, chordal tolerance, etc. limits will be adjusted or violated.
  - In order to facilitate applying an override, the values (velocity, jerk, etc) that are coming from the incoming gcode will be stored in the planner buffer unmodified, then the configured limitations for each axis will be applied and the resulting values will stored as well. This allows for the originally requested values to be adjusted, and the limitations to be honored, without a large cost for computation.
- When an overrider adjustment comes it (from the system managing the delay and override ramping), it will be applied to the planner buffer starting immediately after the currently running block of gcode. All moves from that point on in the plan will be adjusted.
  - If the currently running block of gcode is long enough that the computation required won't cause an issue, then the current block of gcode may be split and the latter portion will be adjusted.
- No guarantees are made about the time between an override request and when it's applied. An override to a lower velocity is not to be considered a "safety" feature or to be expected to function in time to remove a dangerous situation.
