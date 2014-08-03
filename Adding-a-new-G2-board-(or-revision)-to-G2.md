There are three different ways to add a new board or revision to G2:

1. Adding a new revision of an already existing board, such as G2v9.
1. Adding a new shield to an already existing base board, such as the Due.
1. Adding a new board that uses one of the supported platforms. (Currently, this is SAM3X8E and SMA3X8C.)

_Note about naming: In G2 there are `PLATFORM`, `BASE_PLATFORM`, and `MOTATE_BOARD`,  where `PLATFORM` is the top-level that is selected when compiling, and the rest are implied based on that setting. In newer versions of Motate, these have been rearranged and clarified to make more sense: `BOARD`, `BASE_BOARD`, `PLATFORM`, and `ARCH`, where `BOARD` is the top-level that is passed when compiling. This document will be changed to reflect those changes when they happen. Until then, accept that the naming is somewhat nonsensical._

##Adding a new revision of an already existing board.

There are two steps to adding a new revision of an already existing board.

1. Add the new 'platform' to the main Makefile.
1. Duplicate and alter the appropriate pin assignment file.

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

Further down in the makefile you'll see `ifeq ("$(BASE_PLATFORM)","v9_3x8c")` clauses that will contain a addition to `DEVICE_INCLUDE_DIRS` (it will likely refer to `PLATFORM_BASE`, which should also be defined in that clause).

For example, for `BASE_PLATFORM` of `v9_3x8c` the resulting value is `platform/atmel_sam//board/v9_3x8c`

In that directory will be a `motate_pin_assignments.h` and one or more `XYZ-pinout.h` files.

1. In G2, the `motate_pin_assignments.h` file defines all of the constants used throughout the G2 project, and it `#include`s the `${MOTATE_BOARD}-pinout.h` file to get the actual mapping of the Motate pin numbers to their hardware Port/Pin assignments and related functions. This means many boards will use the *same pin numbering and constants*.

  If you are *adding* new pin types to the project, then you must add the constants to `motate_pin_assignments.h`. New functions should be over 100, and must be below 253. It's okay if only one board uses that function (and has a definition for that number), Motate will simply generate a Null pin that can be checked for in the code if necessary. To reserve a name for a pin that no boards define yet, just give it the value -1.

  ```c++
    // Already existing pins:
    pin_number kSD_CardDetectPinNumber          = 119;
    pin_number kInterlock_InPinNumber           = 120;

    // Newly added pin:
    pin_number kNewFunctionPinNumber            = 121;
 ```

  Naming convention: `k` + CamelCasedName_WithUnderscore + `PinNumber`. The underscore is optional and used as a divider between the "group" and "function" portions of the name. For example, `kSocket6_DirPinNumber` - all of the pins that are on socket 6 start with `kSocket6_`.

1. Duplicate the `XYZ-pinout.h` file and rename it to replace `XYZ` with your `PLATFORM` value, resulting in something like `G2v9i-pinout.h`. (Note: The dash *must* be there, and not be an underscore.)

1. Alter the file to have all of the pinned out pins assigned to the correct numbers as assigned in `motate_pin_assignments.h`. (Generating this file is the purpose of the Motate Pinner.)

1. All of the defines for the same pin number must be changed together, since this portion is how all of the Motate subsystems know what pins are capable of what. For example:

  ```c++
  _MAKE_MOTATE_PIN(119, B, 'B', 15);	// SD_CardDetect
    _MAKE_MOTATE_PWM_PIN(119, Motate::PWMTimer<3>, /*Channel:*/ A, /*Peripheral:*/ B, /*Inverted:*/ true); // INVERTED!
  ```

  When changing that pin number `119` in the first line, the `119` on the second line should change to the same value, since it's actually `PORTB 15` that is attached to `PWMTimer<3>` channel `A`. (Note that newer versions of Motate have fixed this to use the Port/Pin to assign the pins functions, simplifying this a fair bit.)

1.  Unassigned pins should be commented out and moved to the bottom of the file.

