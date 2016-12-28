_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

##Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where the entire image can be transmitted to the printer before processing starts. So the protocol needs to support 'printing' while only holding a partial image - in some cases one or more raster lines, in some cases not even a complete line. 

Commercial laser engravers raster at speeds in excess of 200 inches/second (5080 mm/sec). At bit resolutions of 300, 600, 1200 DPI this translates to 60K, 120K and 240K dots per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels would fall short.

The above assumes pixel encoding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve this theoretical bit encoding efficiency are run-length (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade it are greater bit depth or "color channels", non-binary representations such as base64, ASCII85 or ascii-fied numbers, interspersed command and control characters, and other padding or non-image data.

##Design Goals

We refer to a 'render' as a single image rastered onto some surface by a laser cutter operating in engraving mode. A render has 2 phases:

Render setup - set 'header' parameters used by the rending process
Render image - the actual image engraving process
These two specifications may be together in the same file, or may be delivered separately. Obviously the setup must precede the image specification.