## The Problem		
Gcode used for 3D printing and other non-machining applications has gotten messy. Not only is this a problem for portability, compatibility and domain knowledge, but it hinders the multi-modal machines that are starting to emerge.

We want to use Gcode in a way that is as standard as possible but still supports the following:
- All the functions needed by 3d printing, including different processes, materials, multi-head...
- Mixed machines with different types of tools; spindles, cutters, lasers, vacuums, etc...
- Extensibility for specialized applications that does not intrude into the Gcode definitions

<br>More Goals<br>
- Ability to support 3dp functions/extensions that reprap has added over the years
- Provide a more object oriented and managable view of machining resources

<br>Ultimate Goal<br>
- Machine-independent Gcode: be able to print a file on any machine w/o prior knowledge by the slicer, in other words, machine-independent, material-independent slicing. This is what Postscript does for 2d printing. Any postscript file can run on any printer.

## G2 Dialect Summary
- The g2 dialect uses "standard" Gcode, with minimal extensions for 3dp and other applications. "Standard" gcode is determined by examining multiple industry sources and adopting the rough consensus for G and M commands. See [Consensus Gcode](g2dialect-Consensus-Gcode).
- Use Gcode simply as a way to describe the job, not machine setup, configuration, job overrides and other functions. See [g2dialect Operating Model](https://github.com/synthetos/g2/wiki/g2dialect-Operating-Model).
- Use JSON for non-job functions. Provide job extensibility by embedding JSON into Gcode in a way that does not break existing parsers and conventions. See [JSON Active Comments](https://github.com/synthetos/g2/wiki/JSON-Active-Comments)
- Re-cast the Reprap Gcode and Mcodes using the above. See [g2dialect 3D Printing Codes](Configuring-0.99-3D-Printing-Extensions)
