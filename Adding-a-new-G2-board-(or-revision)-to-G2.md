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

###Making a new pin assignment file

