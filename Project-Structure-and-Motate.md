# Project Structure (High-level)

The g2core project consists of two code bases: **g2Core**, and **Motate**.

**g2core** firmware is all of the motion-control, motor-control, communications protocol, etc. portions that represent the "business logic" of the firmware. The g2Core project is housed at http://github.com/synthetos/g2 and doesn't contain any hardware-specific code *except* to wire g2Core-based hardware (such as the TinyG v9, Arduino Due) to the Motate layer.

**Motate** is the interface between the project and the hardware itself. It's designed to be project-agnostic, and even though it's developed in tandem with g2Core, it may be used for other projects as well.

# Project Directory Structure

The directory structure of the g2Core project in brief:

- `./` - *Top level* the directory resulting from a git clone
  - `g2core/` - contains all of the code of the g2Core firmware, excluding what directly deals with the hardware.
    - `board/` - contains all of the board-specific code that is used to inform the rest of the code about available hardware resources, such as how many motor drivers (and what type) , serial ports, USB interfaces, etc. are available.
      - Each directory inside the `board` directory is the name of a board, and contains the code necessary for that hardware.
    - `device/` - contains all of the code for specialized devices, such as step-direction-based stepper motor drivers, NeoPixel driver, etc.
      - Each directory inside `device` is a specific class of device. The board makefiles then add the device directories they need to the compile, and the board-specific code imports and handles integrating these devices with the core code.
      - There should *not* be code directly in the `device` directory. It should all be in subdirectories.
    - `settings/` - contains the files to configure the default settings of the firmware for a specific *machine type*.
      - The *machine type* should not be confused with the *board*. The board is the electronics that drive the machine. These settings are for the machine, and should be as board-agnostic as makes sense for that machine.
      - Settings files have the name pattern of `settings_`*machine type*`.h`
      - Examples: `settings_othermill.h`, `settings_Printrbot_Simple_1608.h`
      - `settings_default.h` is called automatically after the selected settings file, and provides any #defines not already #defined in the selected settings file. The values in the default file result in a machine that is functional (i.e. communicates), but with most active devices disabled.
  - `Motate/` - is a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) that houses a reference to a specific commit of in the Motate repo. ([Here's a nice writeup](https://github.com/blog/2104-working-with-submodules) on submodule on GitHub as well.)
    - The Motate project handles all of the communications directly to the hardware, and allows the g2Core project to be highly portable to other processors.
 - `Resources/` - contains additional, non-code related resources, such as images for documentation, additional debugging helper scripts, etc.

# Where to go from here

Please check out [Getting Started with g2core](Getting-Started-with-g2core).