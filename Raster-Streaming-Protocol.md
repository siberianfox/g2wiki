_This page is for discussion of an efficient laser raster streaming protocol for use with g2core and other CNC controllers capable of driving laser cutters. See [LaserWeb Issue #99](https://github.com/LaserWeb/LaserWeb4/issues/99)_

##Summary
The protocol uses a Gcode canned cycle approach to sending multiple image lines. It uses JSON active comments to provide parameters for the rendering operation. An example of a MVP (minimum viable protocol?) is provided below. The control header is designed to be extensible to accommodate more sophisticated CNC controllers and capabilities. Some options are discussed in the Protocol Extensions below, but are left out of the MVP discussion.

In the example below the JSON is split across multiple lines for readability. In practice all JSON is on unbroken line(s). JSON may also be delivered as multiple unbroken lines if necessary to observe line length constraints by using a G81.2 as a parameter continuation line.

```c++
G81.1 ({"horiz":3000},
       {"vert":2500},
       {"hres":11.811},
       {"vres":11.811},
       {"feed":10000},
       {"over":5.0},
       {"bits":8},
       {"comp":0},
       {"matr":[1,0,0,1,0,0],
       {"chars":254},
       })
;<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!,
;O<DJ+*.@<*K0@<6L(Df-\0Ec5e;DffZ(EZee.Bl.9pF"AGXBPCsi+DGm>@3BB/F*&OCAfu2/AKY
;i(DIb:@FD,*)+C]U=@3BN#EcYf8ATD3s@q?d$AftVqCh[NqF<G:8+EV:.+Cf>-FD5W8ARlolDIa
;l(DId<j@<?3r@:F%a+D58'ATD4$Bl@l3De:,-DJs`8ARoFb/0JMK@qB4^F!,R<AKZ&-DfTqBG%G
;>uD.RTpAKYo'+CT/5+Cei#DII?(E,9)oF*2M7/c~>
G80 (optional)
```

Parameters:

- `horiz` - Horizontal width of the bitmap in pixels (X dimension)
- `vert` - Vertical height of the bitmap in pixels (Y dimension)
- `hres` - Horizontal pixel resolution (X) in pixels per millimeter (PPM)
- `vres` - Vertical pixel resolution (Y) in pixels per millimeter (PPM)
- `feed` - Maximum velocity (F word) for laser movement in mm/minute. The controller will attempt to hit this speed but may run slower to adjust for communications throttling or other machine or runtime limitations. Horizontal scan line steps and traverses will run at machine maximum (G0) and are not specified in this header.

- `over` - Millimeters of horizontal overscan. This is the maximum distance the head will be allowed to travel beyond the horizontal print area to allow for acceleration / deceleration.

- `bits` - Bit depth: Number of bits per pixel - typically 8, for 255 grey levels, but may larger for increased monochrome resolution. A bit-depth of 1 may also be used to allow the rasterizer to perform dithering or other half-toning algorithms. In this case the PPMs may also be set at the dot limit of the laser, typically on the order of 50 to 100 DPM (~1200 - 2400 DPI).

- `comp` - Optional: Compression - Uncompressed bitfield (0) or run-length encoding without Huffman encoding (1). Beyond MVP it may be useful to consider Huffman encoding, X-A delta run-length encoding. and other more advanced encodings as per [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) for further encoding efficiency. See Protocol Extensions. Defaults to uncompressed if omitted.

- `matr` - Optional: The transformation matrix to be applied to the image. In MVP this is merely an XY setting horizontal (X) and vertical (Y) directions from origin. An array value of [1,0,0,1,0,0] sets a raster with a lower left origin. A value of [1,0,0,-1,0,0] flips Y for an upper left origin. Beyond MVP the matrix can be used to perform scaling, rotations, and translation. See  https://www.adobe.com/products/postscript/pdfs/PLRM.pdf, section 4.3.3 for details. Defaults to [1,0,0,1,0,0] (lower left) if omitted. 

- `chars` - Maximum ASCII characters. This parameter allows the rasterizer to tell the controller the maximum number of ASCII characters it will send in an image line (including terminating CR and/or LF characters). If the controller cannot handle this number it should send an error and the number of characters it can handle. (The method of returning the allowable line length is TBD).

`pixel array` - These lines contain up to as many bytes as can fit in a single transmission (ASCII line). The number of characters should not exceed the `chars` value, including terminating LF / CR characters. 

There is not a 1:1 correspondence between ASCII text lines and image lines; An image line may span multiple text lines, and a text line may contain data for 2 or more image lines. Image line breaks are handled by the controller counting pixels, not by looking for ASCII line ends.

The pixel array data is ASCII encoded using [ascii85](https://en.wikipedia.org/wiki/Ascii85). Ascii85 expands binary data 25% (4 binary bytes into 5 ascii bytes) and is used predominantly in PDF and other renderers, as opposed to base64 which expands 33% (3 into 4). As per ascii85, The first line begins with `<~` and the last line ends with `~>`. In this protocol we additionally require that all pixel array lines start with a semicolon comment character `;` for parsing purposes.

The [ZeroMQ (Z85) version of ascii85](https://en.wikipedia.org/wiki/Ascii85#ZeroMQ_Version_.28Z85.29) is recommended, as it is a string-safe variant of base85. By avoiding the double-quote, single-quote, and backslash characters it can be safely carried as a JSON string value, enabling REST or even command line operation of the protocol.

`cancel cycle` - G80. In most cases this command should not be needed as the controller counts pixels based on the image dimensions and terminates the render when complete, and has a "file end" as part of the ascii85 encoding. If a G80 or any other Gcode modal group 1 command (e.g. G0, G1) is encountered during the render the cycle will be terminated at that point. If for some reason the render did not end (e.g. file error), an end can be forced using G80.

#### Notes

1. The origin of the image is at the current location of the tool, and is not specified in the cycle parameters 

# Details
Some vocabulary:

- `Pixel` - a dot in the image - the smallest resolution of the image; e.g. 300 PPI
- `Dot` - a step on the machine - the smallest resolution of the laser engraver machine; e.g. 1200 DPI
- `Pixel Array` - the image data itself
- `Render` or rendering operation - a single raster image lasered onto some surface

## Problem Statement
The challenge is to efficiently transmit a bitmapped image to a CNC controller with limited memory and processing capabilities. If the CNC controller were unconstrained the problem would reduce to look like a regular 2d printer, where all or most of the image fits in memory and can be processed at memory access speeds. So the raster protocol needs to support 'printing' while only holding a very small, partial image - in many cases not even one complete raster line.

At high raster rates and pixel resolution image transmission becomes a bottleneck. Commercial laser engravers can operate at speeds approaching 200 inches/second (5080 mm/sec). At pixel resolutions of 300, 600, 1200 PPI this translates to 60K, 120K and 240K pixels per second. At common communications channels speeds of 115.200 bps, 1 Mbps and 12 Mbps byte transfer speeds are limited to 14,400, 125K and 1.5M bytes / second, assuming 100% channel utilization, which is of course unrealistic. So a fully utilized USB channel should be able to keep up with these transmissions, but slower channels fall short.

Another factor is the efficiency of the pixel data encoding. The pixel rates above assume coding efficiency of 8 bits per pixel, or 255 grey levels (bit depth). Factors can that can improve encoding efficiency are lossless run-length encoding (RLE) and delta run length encoding (DRLE), with or without Huffman encoding. Factors that degrade pixel encoding efficiency include greater bit depth or "color channels", non-binary representations such as base64, ascii85 or asciified numbers, interspersed command and control characters, and other padding or non-image data. Common Gcode pixel methods provide encoding efficiencies less than 10% (10 or more ascii characters per image pixel).

## Design Goals
The goal of the raster protocol is to support laser raster operations as fast as possible given the constraints of typical CNC controllers and communication channels. Design goals are listed here for discussion in no particular order:

- Support Laser/CNC controllers with limited image memory and/or communications bandwidth. This suggests a streaming protocol.

- Consider [lw.rasterizer](https://github.com/lautr3k/lw.rasterizer) as a starting point for protocol generation (Q: is this right, or is there some other program we should be considering?)

- Support a baseline MVP protocol that is as simple as possible, but allow for extensibility for higher efficiencies and controller capabilities. Also allow for an inspection mechanism to allow the CAM to query and utilize capabilities of a given controller (capabilities negotiation).

- Support communications in ASCII only (7 bit) formats as some communication channels are not friendly to raw binary communication. Allow for binary transfer if binary capabilities are available.

- Deal with the case that many image lines will contain more pixel information than can be consumed in a serial line sent to the CNC controller.

- Support interjection of runtime machine controls such as feed rate override, feedhold and resume in the MVP protocol.

- Require minimal translation from common image formats to the streaming format. Specifically, we are concerned with images sources as raw, in-memory bitmaps, [PNG image files](https://en.wikipedia.org/wiki/Portable_Network_Graphics), and [BMP image files](https://en.wikipedia.org/wiki/BMP_file_format), to a lesser extent.

- Approach the bit efficiencies found in raw PNG, BMP or other lossless files, and support simple run-length encoding of raw pixel image lines.

- It may also be an option to support a mode where Gcode is not used at all - e.g. direct REST operation.

#Protocol Extensions
This section is a parking lot for additional things that may be considered beyond MVP functionality.

### On-board Upsampling 
Currently the LW rasterizer performs pixel upsampling - i.e. run N laser passes to get a pixel of the right size given the beam width of the laser. This requires redundant transmission that can be eliminated by moving upsampling operations to the controller.

### Transformation Matrix
Provide full transformation matrix capability for image translation, scaling, rotation and flip. This is an extension of the optional, rudimentary matrix provided in MVP. See: https://www.adobe.com/products/postscript/pdfs/PLRM.pdf, section 4.3.3 for details.

### Arbitrary Scan Lines 
The MVP protocol only handles horizontal scan lines. This item adds diagonal and arbitrary (curves) scan lines.

### Support Native Dot Resolution
Support a special value for native dot resolution as an option for vres and hres to better support direct rasterization of dithering and special grey scale patterns. A 'dot' is a the smallest resolvable step of the CNC machine.

### Bidirectional Scanning
Introduce controls over unidirectional and bidirectional scans. Unidirectional mode can be preferable to eliminate machine backlash "jaggies" at high pixel resolutions. Bidirectional mode will scan in two directions when this is practical. Some modes are only practical if the controller has sufficient memory to store one or two scan lines, which can be arbitrarily large for large machines and high resolutions. 

### DRLE and Huffman Encoding
Introduce 1D (PNG X-A) delta run length encoding and Huffman encoding for run-length for better image compression.

### 2D Compression Algorithms
Introduce 2d compression such as PNG Up, Average, and Paeth. The latter requires additional buffer storage similar to the scanning and upsampling items.

### Note on Items That May Require Larger Image Buffers
There are two cases for memory constraints: (1) the CNC controller does not have sufficient memory to store an entire image line, (2) it does. For case 2, a Lasersaur with a 48" bed running at 1200 DPI with a bit depth of 16 would require 115,200 bytes to store an image line. Most images cases, however, should fit in a reasonable (4K, 8K, 16K) buffer without tiling the image.

For most operations the controller would not need an entire line, but here are some situations where that is not true:

- Pixel-to-dot upsampling, where multiple passes of the laser are required to achieve a pixel of sufficient size
- Bi-directional scanning - where the pixel in the return scan must be played out from "right to left"
- 2D compression encodings like PNG's Up, Average, and Paeth compression filters (actually require *2* lines)
 