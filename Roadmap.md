## Our Roadmap, in broad strokes

Last updated: *5th June 2018*

### Near term objectives

1. **Close out a lot of the PRs that are in the works**

    - [ ] [#341 Add the gShield .elf and .bin files to our releases page](https://github.com/synthetos/g2/pull/341)
      * Discussion on the best approach here is still in progress.

2. **[#285 - M7 Core](https://github.com/synthetos/g2/pull/285)**

Bring in the code to support the Atmel M7 core [Verion 1.0.0](#changes-planned-for-version-100), so it’s part of the main project rather than an offshoot living in the dev-168 branch. That sets the stage for releasing the [gQuintic board](gquintic-specs), which we are almost ready to do. It also gives us an enormous amount of cycles to do some really cool stuff like advanced kinematics. This also has a cleaned up Motate and some stepper enhancements.

3. **[Issue #347 - GPIO Enhancements](https://github.com/synthetos/g2/issues/347)**

    Bring in [refactored GPIO with a bunch of enhancements](gpio-design-discussion). The code is being tested on dev-168 and slated for release in 1.0.0. The CAN stuff slots in after this is available, as the new GPIO model supports all kinds of “non-io IO” like CAN.

## Changes To Branching and Release
[Described here](branching-and-release)

## Changes Planned for Version 1.0.0 
Once the new branching and versioning is in place a merged branch labeled `1.0.0` will be pushed. This combines the work that has been done on edge (the ARM M3 code base) with the work done in the ARM M7 code base. They will then be a single code base with the ARM chip being selected by the board hardware.

This section lists the changes that may be incompatible with previous releases (either branch). This is an attempt to get these all together for the 1.0.0 release and take the hit at one time.

### ID Attribute Changes
Proposed changes to IDs:
* {fv:n} string in form Major.Minor.Patch, e.g. `1.0.0`, not a float
* {fb:n} string with ShortSha, e.g. `49b671a`, not a float

The following remain the same:
* {fbc:n} settings file - e.g. settings_axidraw_v3.h
* {hp:n} board platform string - e.g. gQuintic
* {hv:n} board version string - e.g. revF
* {id:n} board ID string - e.g. 0271-6fd3-e29c-14d

Note: These are and remain Read Only attributes. TinyGv8 has a case where hardware version can be written. This in not so in g2core.

### GPIO Configuration Changes
GPIO configuration is different - See [GPIO Design Discussion](GPIO-Design-Discussion/_edit). Key Changes:
* Mode (mo) has been removed, replaced by:
  * [{...en:n} Enable Setting](gpio-primitives#enn-enable-setting): used to enable and disable IO channels
  * [{...po:n} Polarity Setting](gpio-primitives#pon-polarity-setting): used to set polarity of IO channels
* Function (fn) has been removed from digital inputs:
  * [{di1ac: Action](gpio-primitives#digital-input-configuration-values): handles all actions and functions
  * [{di1in: Exposed-As](gpio-primitives#digital-input-configuration-values): allows flexible bindings of input readers to arbitrary digital input channels

### GPIO Polarity Changes
In addition to the above GPIO configuration changes, all polarities will obey 0=ACTIVE_HI (non-inverted polarity), 1=ACTIVE_LO (inverted polarity). This affects: 
* Coolant
  * {comp:n} coolant mist polarity - 0=active HI, 1=active LO
  * {cofp:n} coolant flood polarity - 0=active HI, 1=active LO
* Spindle
  * {spep:n} spindle enable polarity - 0=active HI, 1=active LO
  * {spdp:n} spindle direction polarity - 0=clockwise HI, 1=clockwise LO

The following remain the same:
* Motor direction polarity {1po:n}
* Motor enable polarity {1ep:n}
* Motor step polarity {1ep:n}

### Changed References to NORMALLY_OPEN and NORMALLY_CLOSED
These are synonymous with the following, and have been replaced by their synonym:
* NORMALLY_OPEN is replaced by IO_ACTIVE_LOW
* NORMALLY_CLOSED is replaced by IO_ACTIVE_HIGH

### Global Motor Enable and Disable Commands (me/md) Changes
Motor enable {me:n} and disable {md:n} commands will function as so:
* {me:n} global motor enable
    * Returns True if any motor is enabled
    * In previous releases this command would enable all motors
* {md:n} global motor disable
    * Returns True if all motors are disabled
    * In previous releases this command would disable all motors

The following remain the same:
* {me:0} enable all motors
* {md:0} disable all motors
* {me:M} enable motor M (where M is a number from 1 to max motor)
* {md:M} disable motor M (where M is a number from 1 to max motor)
* {mt:XXX} sets motor timeout in seconds which is obeyed by enabled motors

Why?: The GET semantic will be changed as it violates RESTful GET safety - GETs should not cause any action. G2core has (and will continue to have) one exception to this rule: that's for clearing an alarm or shutdown condition using `{clear:n}` or `{clr:n}`. In alarm, shutdown and panic conditions no SET commands are accepted - i.e. commands with other than NULL as an argument. In order to accept a clear we make an exception for alarm and shutdown. Panics can only be cleared by a board reset.

### Remove Queue Reports {qr:n}
Queue reports are an early strategy for communications synchronization that has been deprecated in favor of [line mode protocol](g2core-Communications#line-mode-protocol). We are planning to remove Queue reports as of 1.0.0. If this impacts you negatively please contact us. 

### Deprecating G28.2, G28.3 and G28.4 Homing for JSON Versions
Back in the day we added G28.2, G28.3 and G28.4 homing behaviors. These are not strictly Gcode and should never be called from within a Gcode job. We plan to substitute the following JSON commands for these functions:
* G28.2 --> {home:...tbd}
* G28.3 --> {home:...tbd}
* G28.4 --> {home:...tbd}

The G28 forms will be removed in a later release.

## Still To Go
**Clean up the Configs**

* Once we are done with the above there are a series of changes to the configs we want to make to break it up and make it more modular.

* We've also built a machine-builder UI that configures itself based on metadata provided by system objects (think dynamic binding at the HW level). We need the refactoring to get the metadata in play. We have a UI working for this that Rob and our colleague Laurie Barth are presenting at the Nation JS developer conference later this month. But so far we are only driving this from “canned” descriptors, not directly off the hardware.