This page describes system diagnostics that are available in G2

### Diagnostic Groups
The following diagnostic groups can be enabled at compile time. The main use of these groups is to include them in status reports for planner diagnostics.

Group | Tokens | Notes
--------|----------|-------
_te | x y x a b c | target endpoint (in work space, i.e. axis coordinates)
_tr | x y x a b c | target runtime (in work space)
_ts | 1 2 3 4 5 6 | target steps (in joint space, i.e. motor steps)
_ps | 1 2 3 4 5 6 | position_steps
_cs | 1 2 3 4 5 6 | commanded steps (delayed)
_es | 1 2 3 4 5 6 | encoder_steps (measured)
_fe | 1 2 3 4 5 6 | following_error (as read by encoders)
_xs | 1 2 3 4 5 6 | corrected_steps (cumulative correction applied)
