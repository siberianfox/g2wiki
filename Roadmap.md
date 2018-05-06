## Our Roadmap, in broad strokes

Last updated: *5th May 2018*

### Near term objectives

1. **Close out a lot of the PRs that are in the works**

    In rough sequence order:

    - [x] [#307 Spring Compensation](https://github.com/synthetos/g2/pull/307)
      * Just closed it (canceled) as it needs work.
      * We’ll bring that one back up later (after some project stuff, it'll be a lot cleaner).

    - [x] [#334 Update A, B, C axes radius defaults to use motors 4, 5, & 6](https://github.com/synthetos/g2/pull/334)
      * Just approved it.
      * We should go ahead and merge it too.

    - [ ] [#341 Add the gShield .elf and .bin files to our releases page](https://github.com/synthetos/g2/pull/341)
      * Will not conflict.
      * Will review it and pull it asap. Needs Travis review.

    - [x] [#339 Fix coolant-init](https://github.com/synthetos/g2/pull/339)
      * Looks relatively straightforward

    - [ ] [#299 Reading Nested JSON fail](https://github.com/synthetos/g2/pull/299)
      * Waiting for @aldenharts' review and regression testing the JSON command lines

    Then:

    - [ ] [#329: Finish Merge of #320 to Edge](https://github.com/synthetos/g2/pull/329)
      * It changes config app a lot, so we want to do this last (see item #4, below)

2. **M7 Core**

    Bring in the code to support the Atmel M7 core, so it’s part of the main project rather than an offshoot living in the dev-168 branch.

    That sets the stage for releasing the gQuintic board, which we are almost ready to do. (One more proto revision!).

    It also gives us an enormous amount of cycles to do some really cool stuff like advanced kinematics.

    This also has a cleaned up Motate and some stepper enhancements.


3. **GPIO Enhancements**

    Bring in refactored GPIO with a bunch of enhancements. Alden will be posting a wiki page he and Rob have been working on with the details, over the weekend. This is something we have been working on for quite a while.

    The code is in the works but is not completely done or tested yet. The CAN stuff slots in after this is available, as the new GPIO model supports all kinds of “non-io IO” like CAN.


4. **Clean up the Configs**

    Once we are done with the above there are a series of changes to the configs we want to make to break it up and make it more modular.

    We've also built a machine-builder UI that configures itself based on metadata provided by system objects (think dynamic binding at the HW level). We need the refactoring to get the metadata in play.

    We have a UI working for this that Rob and our colleague Laurie Barth are presenting at the Nation JS developer conference later this month. But so far we are only driving this from “canned” descriptors, not directly off the hardware.