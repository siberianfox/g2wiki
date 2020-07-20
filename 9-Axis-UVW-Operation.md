## Functionality
As of firmware build `{fb:101.00}` a limited form of 9 axis motion control has been added to g2. The U, V and W axes have been added as secondary linear axes corresponding to X, Y and Z, respectively. Externally visible axis numbers have been extended to accommodate UVW:

### Axis Mapping

Axis | Axis # | Description (by convention)
------|---| ---------
X | 0 | +X motion moves left to right when facing the machine
Y | 1 | +Y motion moves front to back when facing the machine
Z | 2 | +Z motion moves up to the top when facing the machine; Z is by convention the tool-holding axis (cutting or actuating)
A | 3 | +A rotates clockwise around X as viewed from the positive end (right side of the machine)
B | 4 | +B rotates clockwise around Y as viewed from the positive end (back side of the machine)
C | 5 | +C rotates clockwise around Z as viewed from the positive end (top of the machine)
U | 6 | U is a secondary X axis and moves by the rules of the X axis
V | 7 | V is a secondary Y axis and moves by the rules of the Y axis
W | 8 | W is a secondary Z axis and moves by the rules of the Z axis

### Using UVW Axes
Motors can be mapped to any of the 9 axes. This allows a machine to have as many as six linear axes that are independent of each other and obey coupled kinematics.

The primary use case for the UVW axes are to support a secondary 'Z' axis. Some machines have two toolheads - One can be operated as Z, the other as W. 

### Specs and Limitations

* Coordination Between XYZ and UVW Axes
There is no coupling between the X/U, Y/V or Z/W axis pairs. I.e. they define independent linear axes. Movement in X will not change position in U, or vice versa.

* The feedhold actions support Z lift only - they do not currently support movement in W as a feedhold action.

# Implementation Notes 
(Only useful if you are actually working in the code)
## Internal and External Axis Representation
Legacy axis numbering is XYZABC as 0 through 5. Adding UVW by convention extend this to XYZABCUVW as 0 through 8.

This mapping is provided in g2core.h as the X_AXIS_EXTERNAL - W_AXIS_EXTERNAL enumeration. Settings files must use the external forms (as if they were being provided from an external interface).

Internally the 6 element axis array has been expanded to 9 elements ordered as [X,Y,Z,U,V,W,A,B,C]. This allows the lower 6 positions to carry the linear axes and the upper 3 the rotary axes. The main array can then be split into indexable linear and rotary sub-arrays.

Mapping between external and internal forms has been provided for data entry and display via the axis mapping set and get functions, which is the only place the API actually addresses axes by number. In the future we could make the AM parameter work to letter to make this even more transparent.
