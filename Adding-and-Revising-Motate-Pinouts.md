This page explains how to create or change Motate in assignments and mappings for a board. Please read Adding and Revising Boards first:

1. [Adding and Revising Boards](Adding-and-Revising-Boards) _(please read this first)_
1. [Adding and Revising Configurations and Settings](Adding-and-Revising-Configurations-and-Settings) _(please read this first)_
1. Adding and Revising Motate Pinouts _(this page)_
1. [Adding Configurations to an IDE](Adding-Configurations-to-an-IDE)

## A Brief Background on Motate Pins

All of the hardware is connected through Motate, and Motate uses a single ("flat") numbering scheme for all pins, regardless of what hardware port they are on.

In order to use pins and things that eventually use pins the code will (directly or indirectly) include the `MotatePins.h` file. In that file, it will include `motate_pin_assignments.h` in order to get the assignments of numbers.

The `motate_pin_assignments.h` is provided as part of the project (g2core), and in g2core we have one for each supported hardware "board." They are located in a `board/${BASE_BOARD}/` directory.

In order to add the `board/${BASE_BOARD}/` to the include path so `#include "motate_pin_assignments.h"` will find the right one, you add it to `BOARD_PATH` in the appropriate `board/${BASE_BOARD}.mk` file, and then add `${BOARD_PATH}` to the `SOURCE_DIRS` value. For example, this is from the `g2v9.mk` file:

```makefile
ifeq ("$(BASE_BOARD)","g2v9")
// ... other important stuff ...

BOARD_PATH = ./board/g2v9
SOURCE_DIRS += ${BOARD_PATH} device/step_dir_driver

// ... other important stuff ...
endif
```

### Making a New Pin Assignment

The `motate_pin_assignments.h` file _eventually_ provides two things:

1. Constant names (of type `pin_number`) with unique numbers for use throughout the project.
  - Pins that will be referenced in the code but do not have physical pins can be assigned a value of `-1`.
  - This step is not strictly necessary, but is highly recommended to make the code much more flexible across multiple boards. It is possible to use the Pin objects with number directly, but generally we will not accept code in g2core that refers to pin numbers directly.
  - An example:
  ```c++
  pin_number kOutput3_PinNumber = 132;
  ```

2. Assignment of Motate "pin numbers" to physical pads on the chip. This is handled through macros for simplicity.
  - An example (explained in detail later):
  ```c++
  _MAKE_MOTATE_PIN(kOutput3_PinNumber, 'A', 5);
  ```


Now, while this part isn't required, in order to support revisions of the boards, we include `${MOTATE_BOARD}-pinout.h` which contains the actual `_MAKE_MOTATE_PIN` calls for that revision of the board. (`MOTATE_BOARD` is generally set to `BOARD` in the `board/${BASE_BOARD}.mk` file. It it separated in case your board revisions use a different naming than your `BOARD` files.)

Example last few lines of `motate_pin_assignments.h`:
```c++
#ifdef MOTATE_BOARD
#define MOTATE_BOARD_PINOUT < MOTATE_BOARD-pinout.h >
#include MOTATE_BOARD_PINOUT
#else
#error Unknown board layout $(MOTATE_BOARD)
#endif
```

And now you create one `${MOTATE_BOARD}-pinout.h` for each revision of your board, each with the `_MAKE_MOTATE_PIN` defines and any other defines you need for that board.

A few additional notes:

1. If you are *adding* new pin types to the project that are used _outside_ the `board/` directory, then you must add the constants to `motate_pin_assignments.h` of **all** boards in the project that might also use that constant, or they will no longer compile.
  - It's okay if only one board uses that function and has a definition for that number. Any board that doesn't use that function can have it defined as `-1`.
  - If _and only if_ you are only using the pin constant _inside_ the `board/` directory for your board, then the other boards don't need that constant defined.

  ```c++
    // Already existing pins:
    pin_number kSD_CardDetectPinNumber          = 119;
    pin_number kInterlock_InPinNumber           = 120;

    // Newly added pin:
    pin_number kNewFunctionPinNumber            = 121;
 ```

1. Naming convention: `k` + *`CamelCasedName_WithUnderscore`* + `PinNumber`. The underscore is optional and used as a divider between the "group" and "function" portions of the name. For example, `kSocket6_DirPinNumber` - all of the pins that are on socket 6 start with `kSocket6_`.

1.  Pin sub-functions (PWM, ADC, etc) are assigned in Motate already, but use the port and pin number (`"B", 15` for example) to find the Motate number (`119`), and will cause a compile-time-error if there is no number assigned for that port. So, all pins that are software accessible (have a port and pin, not GND, VCC, etc) *must* have a number assigned, even if it's never used in the software or assigned a constant in the `${MOTATE_BOARD}-pinout.h` files. For those, we have conventionally used constant `kUnassigned`*`XX`* with XX starting from 1 and going up, with values counting down from `254` (the maximum value). Example:
  ```c++
  pin_number kUnassigned2  = 253;
  pin_number kUnassigned1  = 254;  // 254 is the max.. Do not exceed this number
  ```

1. Any newly created pins can now be used in G2 by the new constant name.


## Notes for base-board + shield configuration BOARDs

The order of assigning the numbers in `motate_pin_assignments.h` and then assigning them to physical pads in the `${MOTATE_BOARD}-pinout.h` file is reversed is cases such as the Arduino Due, where the pins are clearly numbered already and many boards will assign _names_ to the _numbers_.

All of the above still holds true, except that the `motate_pin_assignments.h` holds the `_MAKE_MOTATE_PIN(...)` calls using numbers, and the `${MOTATE_BOARD}-pinout.h` holds the `pin_number kBlah = 123;` assignments.

We **do not** recommend using this layout unless the board is designed by someone else (for example a dev board like an Arduino) and has clearly labeled pin numbers for the same physical pads, *and* there are multiple "shields" (etc) that will use that same clearly labeled pin numbering.
