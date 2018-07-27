[Universal G-Code Sender](https://github.com/winder/Universal-G-Code-Sender) (or UGS) is a Java based, cross platform G-Code sender, compatible with GRBL and TinyG/g2core. 

![Screenshot](http://winder.github.io/ugs_website/img/platform/screenshot.png)

## Features
* Cross platform, tested on Windows, OSX, Linux, and Raspberry Pi
* G-code sender with duration estimates
* Configurable macros, keyboard shortcuts and window layout
* 3D Gcode Visualizer with color coded line segments and real time tool position feedback
* Continuous jogging
* G-code state reporting
* Configurable G-code optimization:
  * Remove comments
  * Truncate decimal precision to configurable amount
  * Convert arcs (G2/G3) to line segments
  * Remove whitespace

## Getting started
* Download and install [Java 8](https://java.com/en/download/manual.jsp) or later
* Download the latest nightly build of [Universal G-Code Sender](http://winder.github.io/ugs_website/download/#nightly-builds)
* Go to the `ugsplatform/bin` directory
  * On Windows run `ugsplatform.exe` or `ugsplatform64.exe`
  * On Linux or Mac OSX run `ugsplatform`