_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

##Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where the entire image can be transmitted to the printer before processing starts. So the protocol needs to support 'printing' while only holding a partial image - in some cases one or more raster lines, in some cases not even a complete line. 

Commercial laser engravers raster at speeds in excess of 200 inches/second (5080 mm/sec). At bit resolutions of 300, 600, 1200 DPI this translates to 60K, 120K and 240K dots per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels fall short.

The above assumes dot encoding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve bit encoding efficiency are lossless run-length encoding (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade pixel encoding efficiency include greater bit depth or "color channels", non-binary representations such as base64, ASCII85 or asciified numbers, interspersed command and control characters, and other padding or non-image data. Common Gcode representations degrade by a factor of 10:1 or worse. 

###Design Goals
Here for discussion and listed in no particular order:

- Support Laser/CNC controllers with limited image memory and/or communications bandwidth. This suggests a streaming protocol.

- Support a baseline MVP protocol that is as simple as possible (minimum viable protocol?), but allow for extensibility for additional capabilities a controller may have. Also allow for an inspection mechanism to allow the CAM to query and utilize capabilities of a given controller (capabilities negotiation).

- Support communications in ASCII only (7 bit) formats as some communication channels are not friendly to raw binary communication. Allow for binary transfer if binary capabilities are available.

- Support interjection of runtime machine controls such as feed rate override, feedhold and resume in the MVP protocol.

- Require as little translation from common image formats to streaming protocol formats as possible. Specifically, we are looking at [BMP](https://en.wikipedia.org/wiki/BMP_file_format) and [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) image file formats.

- Ideally we could stream BMP or PNG image files directly to the controller, taking advantage of compression methods already used in those files.

##Protocol Design
Some vocabulary - mostly taken from the BMP and PNG formats

- `Image` - the image data itself
- `Image header` - metadata describing the image, but not the rendering operation
- `Render` or rendering operation - a single raster image lasered onto some surface
- `Render header` - metadata describing the rendering operation, but not the image

### MVP Protocol
The MVP protocol is limited to the following assumptions

- The image will be rendered in a perfect X/Y grid without skew or rotation. No Z movement is possible.

The protocol consists of three parts that are sent as different data elements in the following order

1. Render Header - defines the parameters for the render, including:
  - Starting location of the image (image corner)
  - Unit vector setting horizontal (X) and vertical (Y) directions from start (values must be +1 or -1)
  - 

###Full Protocol
