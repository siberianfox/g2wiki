_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

##Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where the entire image can be transmitted to the printer before processing starts. So the protocol needs to support 'printing' while only holding a partial image - in some cases one or more raster lines, in some cases not even a complete line. 

##Design Goals

We refer to a 'render' as a single image rastered onto some surface by a laser cutter operating in engraving mode. A render has 2 phases:

Render setup - set 'header' parameters used by the rending process
Render image - the actual image engraving process
These two specifications may be together in the same file, or may be delivered separately. Obviously the setup must precede the image specification.