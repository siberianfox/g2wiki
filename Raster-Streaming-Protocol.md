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

- Get as close to the bit utilization in the raw BMP or PNG file as possible. Ideally we could stream BMP or PNG image files directly to the controller, taking advantage of compression methods already used in those files.

- it may be an option to support a mode where Gcode is not used at all - e.g. direct REST operation.

##Protocol Design
Some vocabulary - mostly taken from the BMP and PNG formats

- `Image Header` - metadata describing the image, but not the rendering operation
- `Pixel Array` - the image data itself
- `Render Header` - metadata to set up the rendering operation
- `Render` or rendering operation - a single raster image lasered onto some surface

### MVP Protocol
The MVP protocol is limited to the following assumptions

- The image will be rendered in a perfect X/Y grid without skew or rotation. No Z movement is possible
- Only supports BMP files, and only those not using pixel array compression

The protocol consists of four parts that are sent as different data elements in order. This could be packaged as a canned cycle, or a "gcodeless" option like a REST API.

1. Render Header - defines the setup parameters for the render, including:
  - Starting location of the image (image corner)
  - Unit vector setting horizontal (X) and vertical (Y) directions from start (values must be +1 or -1)
  - Direction - unidirectional or bidirectional (unidirectional to eliminate machine backlash at high bit resolutions)
  - Overscan amount - mm in X
  - Maximum velocities - Separate `F words` for X and Y movement. Controller may run slower to adjust for comm or other limitations 
  - Maximum image line characters. [3]

1. Image Header - Image metadata. Should come from file header and be immutable:
  - Width of the bitmap in pixels (X dimension)
  - Height of the bitmap in pixels (Y dimension)
  - Number of bits per pixel - typically 8, but may be 16 for increased resolution [1]
  - Compression - MVP uses BI_BITFIELDS only, no pixel array compression supported 
  - Size of the raw bitmap data (including padding) [2]
  - Horizontal resolution (X) in pixels/meter (Multiply DPI by 39.3701)
  - Vertical resolution (Y) in pixels/meter (Multiply DPI by 39.3701)

1. Pixel Array - Image contents
  - Sent one line at a time, or as partial lines if line lengths or memory constraints dictate

1. Render Complete - A command that indicates that the render is complete
  - TBD - something like a Gcode G80 to end a canned cycle

Notes:

- [1] We wanted to keep to BMP and PNG standard bit depths, which are 1, 2, 4, 8, 16, and 32 

- [2] We're not sure we need the size of the raw bitmap data, but it's available in BMP and could be useful

- [3] The CAM tells the controller the maximum number of ASCII characters it will send in an image line. If the controller cannot handle this amount it should send an error and the number it can handle.
 

###Full Protocol
