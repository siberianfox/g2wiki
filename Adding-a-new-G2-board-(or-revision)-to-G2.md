There are three different ways to add a new board or revision to G2:

1. [Adding a new revision of an already existing board like the G2v9](#adding-a-new-revision-of-an-already-existing-board).
1. [Adding a new shield of the Due](#adding-a-new-shield-to-an-already-existing-base-board).
1. [Adding a new board that usess one of the supported platforms](#adding-a-new-board-that-uses-one-of-the-supported-platforms). (Currently, this is SAM3X8E and SMA3X8C.)

_Note about naming: In G2 there are `PLATFORM`, `BASE_PLATFORM`, and `MOTATE_BOARD`,  where `PLATFORM` is the top-level that is selected when compiling, and the rest are implied based on that setting. In newer versions of Motate, these have been rearranged and clarified to make more sense: `BOARD`, `BASE_BOARD`, `PLATFORM`, and `ARCH`, where `BOARD` is the top-level that is passed when compiling. This document will be changed to reflect those changes when they happen. Until then, accept that the naming is somewhat nonsensical._

##Adding a new revision of an already existing board.

There are two steps to adding a new revision of an already existing board based on the `v9_3x8c` layout. There is a third step if you want to add it to an Atmel Studio 6 project.

_This is different than adding a new shield layout to the Due, which is described later._

1. Add the new 'platform' to the main Makefile.
1. Duplicate and alter the appropriate pin assignment files; e.g. `TinyG2/platform/atmel_sam/board/v9_3x8c/G2v9k_pinout.h`
1. Add the new config to Atmel Studio 6
  - Menu: Build / Configuration Manager - add the new configuration, e.g. `g2ref_revA`
  - Change the PLATFORM in the build command line in the Build panel of the Properties window 
  - Open AS6, navigate to the `TinyG2/platform/atmel_sam/board` directory and Add New Folder board family name e.g. `g2ref`
  - Copy your starting ...pinout.h and motate_pin_assignments.h file into this directory using Add Existing Items
  - Edit these files. Be sure to save of close AS6 so this sticks

###Add the new 'platform' to the main Makefile.

In the main makefile you will see several clauses like the following:

  ```makefile
  ifeq ("$(PLATFORM)","NewBoardName")
  
  BASE_PLATFORM=v9_3x8c
  DEVICE_DEFINES += MOTATE_BOARD="NewBoardName"
  
  endif
  ```

Make a new clause, using an existing base platform, and change `NewBoardName` to the name of your board. (Naming rules: no spaces, CamelCaseFirstCaps. Preferably matching the naming scheme of other boards, such as `G2v9x`, which `x` changing.)

###Making a new pin assignment

In the `platform/atmel_sam/board/v9_3x8c` directory will be a `motate_pin_assignments.h` and one or more `XYZ-pinout.h` files.

For the `v9_3x8c`, the `motate_pin_assignments.h` file defines all of the constants used throughout the G2 project, and it `#include`s the `${MOTATE_BOARD}-pinout.h` file to get the actual mapping of the Motate pin numbers to their hardware Port/Pin assignments and related functions. This means many boards will use the *same pin numbering and constants*. (Note that the Due uses the same file naming scheme, but the usage is reversed since the Due pinout is set, but the shields each have different constant values.)

1. If you are *adding* new pin types to the project, then you must add the constants to `motate_pin_assignments.h`. New pins should be over 100, and must be below 253. It's okay if only one board uses that function and has a definition for that number. To reserve a name for a pin that no boards define yet, just give it the value -1.

  ```c++
    // Already existing pins:
    pin_number kSD_CardDetectPinNumber          = 119;
    pin_number kInterlock_InPinNumber           = 120;

    // Newly added pin:
    pin_number kNewFunctionPinNumber            = 121;
 ```

  Naming convention: `k` + `CamelCasedName_WithUnderscore` + `PinNumber`. The underscore is optional and used as a divider between the "group" and "function" portions of the name. For example, `kSocket6_DirPinNumber` - all of the pins that are on socket 6 start with `kSocket6_`.

  **Important!** All boards of all kinds *must* have the same pin constants defined, even if they aren't used. Known unused pins should have the value of `-1`. If you add constants to the `v9_3x8c` file, you must also follow the directions [below](#adding-a-new-shield-to-an-already-existing-base-board) to add the same named constant to the existing `Due` files.

1. Duplicate the `XYZ-pinout.h` file and rename it to replace `XYZ` with your `PLATFORM` value, resulting in something like `G2v9i-pinout.h`. (Note: The dash *must* be there, and not be an underscore.)

1. Alter the file to have all of the pinned out pins assigned to the correct numbers as assigned in `motate_pin_assignments.h`. (Generating this file is the purpose of the Motate Pinner.)

1. All of the defines for the same pin number must be changed together, since this portion is how all of the Motate subsystems know what pins are capable of what. For example:

  ```c++
  _MAKE_MOTATE_PIN(119, B, 'B', 15);	// SD_CardDetect
    _MAKE_MOTATE_PWM_PIN(119, Motate::PWMTimer<3>, 
                              /*Channel:*/ A,
                              /*Peripheral:*/ B,
                              /*Inverted:*/ true); // INVERTED!
  ```

  When changing pin number `119` in the first line, the `119` on the second line should change to the same value, since it's actually `PORTB 15` that is attached to `PWMTimer<3>` channel `A`. (Note that newer versions of Motate have fixed this to use the Port/Pin to assign the pins functions, simplifying this a fair bit.)

1. Unassigned pins should be commented out and moved to the bottom of the file.

1. Any newly created pins can now be used in G2.

###Adding a new Config to an Atmel Studio Project

1. Open Atmel Studio 6 to the TinyG2 project
1. Go to the Solution Explore and add the new pinouts file to the platform directory by adding an existing item to the board/xxxxx sub-directory 
1. Go to the Configuration Manager in the Build menu item
1. Select `<New...>`, add your new platform, and copy the parameters from one that's close (or not).
1. Bring up the Properties window by right clicking on the TinyG2 root in solution explorer and select Properties (at the bottom)
1. Go to the Build tab and edit the PLATFORM argument for the new platform. Do this for both the Build commandline and the Clean commandline
1. You should now be able to compile using the standard Build compile options

###Compiling from Command Line (Xcode or Linux)

1. To compile, call `make` with `PLATFORM=NewPlatformName` with your new platform name instead of `NewPlatformName`. It *is* case sensitive. For example, to build the `gShield` variant:

  ```bash
  make PLATFORM=gShield
  ```

##Adding a new shield to an already existing base board.

This is almost the same as adding a variant to the `v9_3x8c` line, except that the pin number assignments are static for the Due, and the constants change values based on the functions that are attached to which pins. 

1. Add the new 'platform' to the main Makefile.
1. Duplicate and alter the appropriate pin assignment files.

###Add the new 'platform' to the main Makefile.

Follow the [same instructions](#add-the-new-platform-to-the-main-makefile) for altering the Makefile as for the `v9_3x8c`.

###Making a new pin assignment

In the `platform/atmel_sam/board/due` directory will be a `motate_pin_assignments.h` and one or more `XYZ-pinout.h` files.

For the `due`, the `motate_pin_assignments.h` file defines the actual mapping of the Motate pin numbers to their hardware Port/Pin assignments and related functions for the Arduino Due, and it `#include`s the `${MOTATE_BOARD}-pinout.h` file to get all of the constants used throughout the G2 project. This means many "shields" will use the Due pin assignments and silkscreen numbering, but the *constants will likely have different values*. (Note that the `v9_3x8c` uses the same file naming scheme, but the usage is reversed since the `v9_3x8c` boards will each have different pinouts, but the constants will all have the same values.)


1. Duplicate the `XYZ-pinout.h` file and rename it to replace `XYZ` with your `PLATFORM` value, resulting in something like `gShield-pinout.h`. (Note: The dash *must* be there, and not be an underscore.)

1. Alter the pin constants to match the pin mapping of the new shield, using the silk-screen pin numbers of the Due. If you are *adding* new pin types to the project, then you must add the constants. It's okay if only one board uses that function and has a definition for that number. To reserve a name for a pin that no boards define yet, just give it the value -1.

  ```c++
    // Already existing pins:
    pin_number kSD_CardDetectPinNumber          = 119;
    pin_number kInterlock_InPinNumber           = 120;

    // Newly added pin:
    pin_number kNewFunctionPinNumber            = 121;
 ```

  Naming convention: `k` + `CamelCasedName_WithUnderscore` + `PinNumber`. The underscore is optional and used as a divider between the "group" and "function" portions of the name. For example, `kSocket6_DirPinNumber` - all of the pins that are on socket 6 start with `kSocket6_`.

  **Important!** All boards of all kinds *must* have the same pin constants defined, even if they aren't used. Known unused pins should have the value of `-1`. If you add constants to the `Due` files, you must also follow the directions [above](#adding-a-new-revision-of-an-already-existing-board) to add the same named constant to the existing `v9_3x8c` files.

1. Any newly created pins can now be used in G2.

1. To compile, call `make` with `PLATFORM=NewPlatformName` with your new platform name instead of `NewPlatformName`. It *is* case sensitive. For example, to build the `gShield` variant:

  ```bash
  make PLATFORM=gShield
  ```

## Adding a new board that uses one of the supported platforms.

**ToDo** Complete this section. Use the following notes:

1. Alter the makefile as above.
1. Add a new section like the following:
  ```makefile
  ifeq ("$(BASE_PLATFORM)","v9_3x8c")
    _PLATFORM_FOUND = 1
    FIRST_LINK_SOURCES += motate/SamTimers.cpp motate/SamUSB.cpp motate/SamPins.cpp

    CHIP = SAM3X8C
    PLATFORM_BASE = platform/atmel_sam
    CMSIS_ROOT = CMSIS

    DEVICE_INCLUDE_DIRS += $(PLATFORM_BASE)/board/v9_3x8c

    include $(PLATFORM_BASE).mk
  endif
  ```
1. Duplicate one of the `platform/atmel_sam/board/v9_3x8c` or `platform/atmel_sam/board/due` directories, depending on usage pattern.

1. Alter the new files accordingly.

