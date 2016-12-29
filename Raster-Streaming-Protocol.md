_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

##Summary
The protocol uses a canned cycle approach to sending multiple image lines. It uses JSON active comments to provide parameters for the rendering operation. In the example below the JSON is split across multiple lines for readability. In practice all JSON is on unbroken line(s). JSON may also be delivered as multiple unbroken lines if necessary to observe line length constraints.

```c++
G81.1 ({"horiz":3000},
       {"vert":2500},
       {"hres":300},
       {"vres":300},
       {"feed":10000},
       {"scan":1},
       {"over":5.0},
       {"bits":8},
       {"comp":0},
       {"matr":{"x":1,"y":1},
       {"chars":254},
       })
<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!O<DJ+*.@~>
<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!O<DJ+*.@~>
<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!O<DJ+*.@~>
<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!O<DJ+*.@~>

G80 (optional)
```

Parameters:

- `horiz` - Horizontal width of the bitmap in pixels (X dimension)
- `vert` - Vertical height of the bitmap in pixels (Y dimension)
- `hres` - Horizontal pixel resolution (X) in pixels per inch (PPI)
- `vres` - Vertical pixel resolution (Y) in pixels per inch (PPI)
- `feed` - Maximum velocity is an `F word` for X and Y movement in mm/minute. The controller will attempt to hit this speed but may run slower to adjust for communications throttling or other machine or runtime limitations. Horizontal scan line steps will run at machine maximum (G0) and are not specified in the render header.

- `scan` - Unidirectional (1) or bidirectional-reversed (2). Unidirectional mode can be used to eliminate machine backlash "jaggies" at high pixel resolutions. Bidirectional-reversed will scan in two directions. The rasterizer program is responsible for reversing the pixel ordering in the 'return' lines. [See also note 2].

- `over` - Overscan in the width dimension, as millimeters. Distance the head will travel beyond the print area to allow for acceleration / deceleration to not require compensation.

- `bits` - Bit depth: Number of bits per pixel - typically 8, for 255 grey levels, but may be 16 for increased monochrome resolution. A bit-depth of 1 may also be used to allow the rasterizer to perform dithering or other half-toning algorithms. In this case the PPI may also be set at the dot limit of the laser, typically about 1200 DPI. FYI: PNG and BMP standard bit depths are 1, 2, 4, 8, 16, and 32 (and 64 in some cases).

- `comp` - Compression - MVP uses (0) for uncompressed bitfield, (1) for run-length encoding compression without Huffman encoding. (for MVP). Beyond MVP it may be useful to consider Huffman encoding and X-A delta run-length encoding as per PNG for further encoding efficiency.

- `matr` - The transformation matrix to be applied to the image. In MVP this is merely a unit vector setting horizontal (X) and vertical (Y) directions from origin (See Note 1). Values of 1 or -1 specify straight lines and may be used to accomplish vertical or horizontal flips. Non-integer values are used to specify diagonal scan lines. The unit vector must obey this equality: 1 = sqrt(x^2 + y^2)

- `chars` - Maximum characters. This parameter allows the rasterizer to tell the controller the maximum number of ASCII characters it will send in an image line (including terminating CR and/or LF characters). If the controller cannot handle this number it should send an error and the number of characters it can handle.

`pixel array` - These lines send as many bytes as fit conveniently in a single transmission (ASCII line) as stated in the chars parameter. The text lines do not correspond to raster lines, and they can carry image data that spans multiple lines. Image line breaks are handled by the controller counting pixels, not by looking for line ends.

The pixel array image data is ASCII encoded using [ascii85]((https://en.wikipedia.org/wiki/Ascii85)). ASCII85 expands binary data 25% (4 binary bytes into 5 ascii bytes) and is used predominantly in PDF and other renderers, as opposed to base64 which expands 33% (3 into 4). As per the encoding standard, all lines begin with `<~` and end with `~>`, so no additional line delimiter characters are required.

The ZeroMQ (Z85) version of ascii85 is recommended, as it is a string-safe variant of base85. By avoiding the double-quote, single-quote, and backslash characters, it can be safely carried as a JSON string value, enabling REST or even command line operation of the protocol.

`cycle end` - G80. In most cases this command should not be needed as the controller counts pixels based on the image dimensions and terminates the render when complete. If a G80 or any other Gcode modal group 1 commnad (e.g. G0, G1) is encountered during the render the cycle will be terminated at that point. If for some reason the render did not end (e.g. file error), an end can be forced using G80.

####Notes:

1. The origin of the render is at the current location of the tool, and is not specified in the cycle parameters 

1. Beyond MVP, Bidirectional-straight would be another scan mode where the CNC controller is responsible for reversing the return line. This is only possible if the controller has sufficient memory to store two or more arbitrarily long scan lines. This mode is useful for unpacking PNG Up, Average, and Paeth compression filters.

# Details
##Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where all or most of the image fits in memory and can be processed at memory access speeds. So the raster protocol needs to support 'printing' while only holding a very small, partial image - in many cases not even one complete raster line.

At high raster rates and pixel resolution image transmission becomes a bottleneck. Commercial laser engravers can operate at speeds as fast as of 200 inches/second (5080 mm/sec). At pixel resolutions of 300, 600, 1200 PPI this translates to 60K, 120K and 240K pixels per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels fall short.

Another factor is the efficiency of the pixel data encoding. The pixel rates above assume coding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve encoding efficiency are lossless run-length encoding (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade pixel encoding efficiency include greater bit depth or "color channels", non-binary representations such as base64, ascii85 or asciified numbers, interspersed command and control characters, and other padding or non-image data. Common Gcode pixel methods provide encoding efficiencies less than 10% (10 or more ascii characters per image pixel).

###Design Goals
The goal of the raster protocol is to support laser raster operations as fast as possible given the constraints of typical CNC controllers and communication channels. Design goals are listed here for discussion in no particular order:

- Support Laser/CNC controllers with limited image memory and/or communications bandwidth. This suggests a streaming protocol.

- Consider [lw.rasterizer](https://github.com/lautr3k/lw.rasterizer) as a starting point for protocol generation (Q: is this right, or is there some other program we should be considering?)

- Support a baseline MVP protocol that is as simple as possible (minimum viable protocol?), but allow for extensibility for higher efficiencies and controller capabilities. Also allow for an inspection mechanism to allow the CAM to query and utilize capabilities of a given controller (capabilities negotiation).

- Support communications in ASCII only (7 bit) formats as some communication channels are not friendly to raw binary communication. Allow for binary transfer if binary capabilities are available.

- Deal with the case that many image lines will contain more pixel information than can be consumed in a serial line sent to the CNC controller.

- Support interjection of runtime machine controls such as feed rate override, feedhold and resume in the MVP protocol.

- Require minimal translation from common image formats to the streaming format. Specifically, we are concerned with images sources as raw, in-memory bitmaps, [PNG image files](https://en.wikipedia.org/wiki/Portable_Network_Graphics), and [BMP image files](https://en.wikipedia.org/wiki/BMP_file_format), to a lesser extent.

- Approach the bit utilizations found in raw PNG, BMP or other lossless files, and support simple run-length encoding of raw pixel image lines.

- It may also be an option to support a mode where Gcode is not used at all - e.g. direct REST operation.

##Protocol Design
Some vocabulary:

- `Pixel` - a dot in the image - the smallest resolution of the image; e.g. 300 PPI
- `Dot` - a step on the machine - the smallest resolution of the laser engraver machine; e.g. 1200 DPI
- `Pixel Array` - the image data itself
- `Render` or rendering operation - a single raster image lasered onto some surface

### MVP Protocol
The MVP protocol should cover most common use cases. See Protocol Extensions for a discussion of additional features that may become useful.



###Protocol Extensions
This section is a parking lot for additional things that may be considered beyond MVP functionality.

- Provide a full matrix definition for image translation, scaling, rotation and flip. This is an extension of the XY unit vector into 3 dimensions

- Provide more flexibility in the definition of the scan line. Move in Z, curves.
