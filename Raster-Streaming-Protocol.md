_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

##Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where the entire image can be transmitted to the printer before processing starts. So the protocol needs to support 'printing' while only holding a partial image - in some cases one or more raster lines, in some cases not even a complete line. 

Commercial laser engravers raster at speeds in excess of 200 inches/second (5080 mm/sec). At bit resolutions of 300, 600, 1200 DPI this translates to 60K, 120K and 240K dots per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels fall short.

The above assumes dot encoding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve bit encoding efficiency are lossless run-length encoding (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade pixel encoding efficiency include greater bit depth or "color channels", non-binary representations such as base64, ASCII85 or asciified numbers, interspersed command and control characters, and other padding or non-image data. Common Gcode representations degrade by a factor of 10:1 or worse. 

##Design Goals
Here for discussion and listed in no particular order:

- Support Laser/CNC controllers with limited image memory and/or communications bandwidth. This therefore suggests a streaming protocol.

- Support a protocol that is as simple as possible (MVP, or minimum viable protocol?), but allow for extensibility for additional capabilities ac controller may have, and potentially for a capabilities discovery mechanism to allow the CAM to optimally utilize the capabilities of the controller.

- Support communications in ASCII only (7 bit) formats as some communication channels are not friendly to raw binary communication.

- Support interjection of runtime machine controls such as feedhold and resume.

- Ideally we could stream a [BMP](https://en.wikipedia.org/wiki/BMP_file_format) or [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) image files directly to the controller, taking advantage of compression methods already used in those files.

- Require as little translation from common image formats to streaming protocol formats as possible. Specifically, we are looking at [BMP](https://en.wikipedia.org/wiki/BMP_file_format) and [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) image file formats.

Some vocabulary - mostly taken from the BMP and PNG formats

- `Image` - the image data itself
- `Image header` - metadata describing the image, but not the rendering operation
- `Render` or rendering operation - a single raster image lasered onto some surface
- `Render header` - metadata describing the rendering operation, but not the image

