asdasd_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters._

## Discussion
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where all or most of the image fits in memory and can be processed at memory access speeds. So the raster protocol needs to support 'printing' while only holding a very small, partial image - in many cases not even one complete raster line.

At high raster rates and pixel resolution image transmission becomes a bottleneck. Commercial laser engravers can operate at speeds as fast as of 200 inches/second (5080 mm/sec). At pixel resolutions of 300, 600, 1200 PPI this translates to 60K, 120K and 240K pixels per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels fall short.

Another factor is the efficiency of the pixel data encoding. The pixel rates above assume coding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve encoding efficiency are lossless run-length encoding (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade pixel encoding efficiency include greater bit depth or "color channels", non-binary representations such as base64, ascii85 or asciified numbers, interspersed command and control characters, and other padding or non-image data. Common Gcode pixel methods provide encoding efficiencies less than 10% (10 or more ascii characters per image pixel).

### Design Goals
The goal of the raster protocol is to support laser raster operations as fast as possible given the constraints of typical CNC controllers and communication channels. Design goals are listed here for discussion in no particular order:

- Support Laser/CNC controllers with limited image memory and/or communications bandwidth. This suggests a streaming protocol.

- Consider [lw.rasterizer](https://github.com/lautr3k/lw.rasterizer) as a starting point for protocol generation (Q: is this right, or is there some other program we should be considering?)

- Support a baseline MVP protocol that is as simple as possible (minimum viable protocol?), but allow for extensibility for higher efficiencies and controller capabilities. Also allow for an inspection mechanism to allow the CAM to query and utilize capabilities of a given controller (capabilities negotiation).

- Support communications in ASCII only (7 bit) formats as some communication channels are not friendly to raw binary communication. Allow for binary transfer if binary capabilities are available.

- Support interjection of runtime machine controls such as feed rate override, feedhold and resume in the MVP protocol.

- Require minimal translation from common image formats to the streaming format. Specifically, we are concerned with images sources as raw, in-memory bitmaps, [PNG image files](https://en.wikipedia.org/wiki/Portable_Network_Graphics), and [BMP image files](https://en.wikipedia.org/wiki/BMP_file_format), to a lesser extent.

- Approach the bit utilizations found in raw PNG, BMP or other lossless files, and support simple run-length encoding of raw pixel image lines.

- It may also be an option to support a mode where Gcode is not used at all - e.g. direct REST operation.

## Protocol Design
Some vocabulary:

- `Pixel` - a dot in the image - the smallest resolution of the image; e.g. 300 PPI
- `Dot` - a step on the machine - the smallest resolution of the laser engraver machine; e.g. 1200 DPI
- `Image Header` - metadata describing the image, but not the rendering operation
- `Pixel Array` - the image data itself
- `Render Header` - metadata to set up the rendering operation
- `Render` or rendering operation - a single raster image lasered onto some surface

### MVP Protocol
The MVP protocol should cover most common use cases. See Protocol Extensions for a discussion of additional features that may become useful.

The protocol consists of four parts that are sent as different data elements in order.

1. **Render Header** - defines the setup parameters for the rendering operation
1. **Image Header** - contains metadata about the image itself
1. **Pixel Array** - contains the image contents
1. **Render End** - a command to end the rendering operation

#### Render Header
Elements:

  - Unit vector setting horizontal (X) and vertical (Y) directions from origin (See Note 1). Values of 1 or -1 specify straight lines and may be used to accomplish vertical or horizontal flips. Non-integer values are used to specify diagonal scan lines. The unit vector must obey this equality: 1 = sqrt(x^2 + y^2)

  - Scan - unidirectional (1) or bidirectional-reversed (2). Unidirectional mode can be used to eliminate machine backlash "jaggies" at high pixel resolutions. Bidirectional-reversed will scan in two directions. The rasterizer program is responsible for reversing the pixel ordering in the 'return' lines. [See also note 2].

  - Overscan amount - mm in X that the head will travel beyond the print area to allow for acceleration / deceleration to not require compensation.

  - Maximum velocities - An `F word` for X and Y movement in mm/minute. The controller will attempt to hit this speed but may run slower to adjust for communications throttling or other machine or runtime limitations. Horizontal scan line steps will run at machine maximum (G0) and are not specified in the render header.

  - Maximum characters. This parameter allows the rasterizer to tell the controller the maximum number of ASCII characters it will send in an image line (including terminating CR and/or LF characters). If the controller cannot handle this number it should send an error and the number of characters it can handle.

  - Example JSON representation:
    ```json
{"rhdr":{"unitx":1,"unity":-1,"scan":1,"overs":5.0,"velx":10000,"vely":1000,"chars":254}}
    ```

[Note 1]: The origin of the render is at the current location of the tool, and is not specified in the render header.

[Note 2]: Beyond MVP, Bidirectional-straight would be another scan mode where the CNC controller is responsible for reversing the return line. This is only possible if the controller has sufficient memory to store two or more arbitrarily long scan lines. This mode is useful for unpacking PNG Up, Average, and Paeth compression filters.

#### Image Header
Elements:

  - Width of the bitmap in pixels (X dimension)
  - Height of the bitmap in pixels (Y dimension)
  - Horizontal resolution (X) in pixels per inch (PPI)
  - Vertical resolution (Y) in pixels per inch (PPI)
  - Bit depth: Number of bits per pixel - typically 8, for 255 grey levels, but may be 16 for increased monochrome resolution. A bit-depth of 1 may also be used to allow the rasterizer to perform dithering or other half-toning algorithms. In this case the PPI may also be set at the dot limit of the laser, typically about 1200 DPI. FYI: PNG and BMP standard bit depths are 1, 2, 4, 8, 16, and 32 (and 64 in some cases).
  - Compression - MVP uses (0) for uncompressed bitfield, (1) for run-length encoding compression without Huffman encoding. (for MVP). Beyond MVP it may be useful to consider Huffman encoding and X-A delta run-length encoding as per PNG for further encoding efficiency.

  - Example JSON representation:
    ```json
{"ihdr":{"dimx":1000,"dimy":1000,"resx":300,"resy":300,"bits":8,"comp":0,"size":424242}}
    ```

#### Pixel Array
Send as many bytes as fit conveniently in a single transmission (ASCII line). Image content lines do not need to correspond to raster lines - i.e. pixels for adjacent lines can be carried in the same transmission.

  - Image data is encoded in ASCII using the [ZeroMQ (Z85) version of base-85 (aka ascii85)](https://en.wikipedia.org/wiki/Ascii85). ASCII85 expands binary data 25% (4 binary bytes into 5 ascii bytes) and is used predominantly in PDF and other renderers, as opposed to base64 which expands 33% (3 into 4). The ZeroMQ base-85 encoding algorithm is a string-safe variant of base85. By avoiding the double-quote, single-quote, and backslash characters, it can be safely carried as a JSON string value.

  - Example JSON 93 bytes total (including LF) carrying 64 pixels of image data - about 45% expansion over binary
    ```json
{"i":"<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!O<DJ+*.@~>"}
    ```

  - Example JSON 253 bytes total (including LF) carrying 192 pixels of image data - about 32% expansion over binary
    ```json
{"i":"<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G<+EV:2F!,O<DJ+*.@<*K0@<6L(Dfc0Ec5e;DffZ(EZee.Bl.9pF0AGXBPCsi+DGm>@3BB/F*&OCAfu2/AKYi(DIb:@FD,*)+C]U=@3BN#EcYf8ATD3s@q?d$AftVqCh[NqF<G:8+EV:.+Cf>-FD5W8ARlolDIal(DId<j@<r@:F%a+D5'~>"}
    ```

#### Render End
A command to end the render. In most cases this command should not be needed as the controller counts pixels based on the image dimensions and terminates the render when complete. If the REND is encountered during the render the render will be terminated at that point. If for some reason the render did not end (e.g. file error), an end can be forced.

  - Something like JSON tag `{"rend":true}` or a Gcode G80 to end a canned cycle (TBD)

### Protocol Extensions
This section is a parking lot for additional things that may be considered beyond MVP functionality.

- Provide a full matrix definition for image translation, scaling, rotation and flip. This is an extension of the XY unit vector into 3 dimensions

- Provide more flexibility in the definition of the scan line. Move in Z, curves.
