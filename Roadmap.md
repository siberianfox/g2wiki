## Our Roadmap, in broad strokes

Last updated: *5th June 2018*

### Near term objectives

1. **Close out a lot of the PRs that are in the works**

    In rough sequence order:

    - [x] [#299 Reading Nested JSON fail](https://github.com/synthetos/g2/pull/299)
      * Done

    - [x] [#329: Finish Merge of #320 to Edge](https://github.com/synthetos/g2/pull/329)
      * Done

    - [ ] [#341 Add the gShield .elf and .bin files to our releases page](https://github.com/synthetos/g2/pull/341)
      * Discussion on the best approach here is still in progress.

2. **[#285 - M7 Core](https://github.com/synthetos/g2/pull/285)**

    Bring in the code to support the Atmel M7 core, so it’s part of the main project rather than an offshoot living in the dev-168 branch. That sets the stage for releasing the gQuintic board, which we are almost ready to do. (One more proto revision!). It also gives us an enormous amount of cycles to do some really cool stuff like advanced kinematics. This also has a cleaned up Motate and some stepper enhancements.


3. **[Issue #347 - GPIO Enhancements](https://github.com/synthetos/g2/issues/347)**

    Bring in refactored GPIO with a bunch of enhancements. See discussion [here](gpio-design-discussion). The code is in the works but is not completely done or tested yet. The CAN stuff slots in after this is available, as the new GPIO model supports all kinds of “non-io IO” like CAN.


4. **Clean up the Configs**

    Once we are done with the above there are a series of changes to the configs we want to make to break it up and make it more modular.

    We've also built a machine-builder UI that configures itself based on metadata provided by system objects (think dynamic binding at the HW level). We need the refactoring to get the metadata in play.

    We have a UI working for this that Rob and our colleague Laurie Barth are presenting at the Nation JS developer conference later this month. But so far we are only driving this from “canned” descriptors, not directly off the hardware.