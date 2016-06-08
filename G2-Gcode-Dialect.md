This page contains the background and details of the g2 machine control Gcode dialect

Related Pages
- Operation Model
	
##The Problem		
We want to use Gcode in a way that is as standard as possible but still supports the following:
- All the functions needed by 3d printing, including different processes, materials, multi-head...
- Mixed machines with different types of tools; spindles, cutters, lasers, vacuums, etc... 
- Extensibility for specialized applications that does not intrude into the Gcode definitions

<br>More Goals<br>
- Ability to support 3dp functions/extensions that reprap has added over the years	
- Provide a more object oriented and managable view of machining resources	

<br>Ultimate Goal<br>
- Machine-independent Gcode: be able to print a file on any machine w/o prior knowledge by the slicer, in other words, machine-independent, material-independent slicing. This is what Postscript does for 2d printing. Any postscript file can run on any printer.

##Summary of the Dialect
- The g2 dialect uses "standard" Gcode, with minimal extensions for 3dp and other applications	
- Extensibility is provided by using JSON commands directly, or as JSON carried in active comments within the Gcode	
- Jobs are segmented