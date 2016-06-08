This page contains the background and details of the g2 machine control Gcode dialect

Related Pages
- Operation Model
	
##The Problem		
We want to use Gcode in a way that is as standard as possible but still supports the following:
- The complete set of functions needed by 3d printing, including different processes, materials, multi-head, etc.
- Mixed machines with different types of tool heads, e.g. spindles, cutters, lasers, vacuums, etc. 
- Extensibility for specialized applications that does not intrude into the Gcode definitions

###Other Goals		
- Ability to support 3dp functions/extensions that reprap has added over the years	
- Provide a more object oriented and managable view of machining resources	
- Ultimate Goal:
  - Make it possible for a print file (Gcode) to be used on any machine w/o prior knowledge by the slicer,
  - in other words, machine-independent, material-independent slicing
  - This is what Postscript does for 2d printing. Any postscript file can run on any printer

##Summary of the Dialect
- The g2 dialect uses "standard" Gcode, with minimal extensions for 3dp and other applications	
- Extensibility is provided by using JSON commands directly, or as JSON carried in active comments within the Gcode	
- Jobs are segmented