## Functionality
As of firmware build `{fb:101.00}` a limited form of 9 axis motion control has been added to g2. The U, V and W axes have been added as secondary linear axes corresponding to X, Y and Z, respectively. Externally visible axis numbers have been extended to accommodate UVW:

### Axis Mapping

Axis | Axis # | Description (by convention)
------|---| ---------
X | 0 | X moves left to right when facing the machine; positive motion moving to the right
Y | 1 | Y moves forward to back when facing the machine; positive motion moving to the back
Z | 2 | Z is the tool-holding axis (cutting or actuating), and moves up and down when facing the machine; positive motion moving up
A | 3 | A rotates around X with clockwise as viewed from the positive side (right side)
B | 4 | B rotates around Y with clockwise as viewed from the positive side (back side)
C | 5 | C rotates around Z with clockwise as viewed from the positive side (top)
U | 6 | U is a secondary X axis and moves by the rules of that axis
V | 7 | V is a secondary Y axis and moves by the rules of that axis
W | 8 | W is a secondary Z axis and moves by the rules of that axis

### Using UVW Axes
Motors can be mapped to any of the 9 axes. This allows a machine to have as many as sis linear axes that are independent of each other and obey coupled kinematics.

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
