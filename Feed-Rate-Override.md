**WARNING: This function is experimental and not complete. Expect changes and refinements to the functionality an the interface functions. Check back for updates**
PAGE STARTED: 18-Aug-15 - ash

##Manual Feed Rate Overrides

Notes:
- Requires g2 edge (089.02) or later
- Usage:
  - Enable master overrides `{m48e:1}`
  - Enable manual feed rate override `{mfoe:1}`
  - Enter / adjust manual feed rate override factor `{mfo:N}` where 0.05 < N < 2.00
- Limitations
  - Does not affect moves already int he planner. Only affects incoming moves
  - Does not currently ramp between velocities - relies on jerk to limit rate-of-change

Bugs:
- Please report and bugs or other comments to g2 Issues