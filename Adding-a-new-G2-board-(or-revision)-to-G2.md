There are three different ways to add a new board or revision to G2:

1. Adding a new revision of an already existing board, such as G2v9.
1. Adding a new shield to an already existing base board, such as the Due.
1. Adding a new board that uses one of the supported platforms. (Currently, this is SAM3X8E and SMA3X8C.)

_Note about naming: In G2 there are `PLATFORM`, `BASE_PLATFORM`, and `MOTATE_BOARD`,  where `PLATFORM` is the top-level that is selected when compiling, and the rest are implied based on that setting. In newer versions of Motate, these have been rearranged and clarified to make more sense: `BOARD`, `BASE_BOARD`, `PLATFORM`, and `ARCH`, where `BOARD` is the top-level that is passed when compiling. This document will be changed to reflect those changes when they happen. Until then, accept that the naming is somewhat nonsensical._

##Adding a new revision of an already existing board.

There are two steps to adding a new revision of an already existing board.

###Add the new 'platform' to the main Makefile.

```makefile
ifeq ("$(PLATFORM)","G2v9i")

BASE_PLATFORM=v9_3x8c
DEVICE_DEFINES += MOTATE_BOARD="G2v9i"

endif
```