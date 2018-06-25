# --- DRAFT ---
_**This page discusses proposed changes to the branching and release scheme for g2core. Please feel free to comment and edit.**_

# Current Branch and Release
* **Master branch** - is currently the stable release branch. Pushes to master are performed periodically as code gets sufficient field history from early adopters of edge code
* **Edge branch** - is currently the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* **Issue-NNN branches** - are created to work on bugs or features. These are pulled into edge via pull requests. 
* **Dev branches** - The Dev prefix has already been deprecated, but you may still see some dev branches around.

# New Branch and Release
The proposed scheme is below:
* **Release branch** - is the stable release branch. Pushes to Release are performed periodically as code gets sufficient field history from early adopters of edge code. Some patches may be applied to Release, but these should always be promoted up through Master. New feature development should not be done against Release. 
* **Master branch** - is the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* **Issue-NNN branches** - are created to work on bugs or features. These are pulled into Master via pull requests against Master. 
* **Edge branch** - Edge will be deprecated and ultimately removed. 
* **Dev branches** - The Dev prefix has been deprecated, but you may still see some dev branches around.

## Semantic Versioning
The new branches will work to a fairly standard semantic versioning scheme - major.minor.patch:
* **Major**: Anything can change, including:
  * changes to machining behaviors
  * changes that affect the semantics of existing JSON variables
  * changes that require configuration files to revised
    * e.g. polarity sense
  * minor or patch level changes may also be included, but do not force a Major revision
* **Minor**: Limited things can change, including:
  * new machining or other features
  * new JSON objects can be added
  * new JSON tags can be added to existing objects
    * ...as long as they do not change the semantics of existing attributes
  * new values to existing objects
  * patch level changes may also be included, but do not force a Minor revision 
  * NOTE: In some limited and very well documented cases a deprecated command may be removed that may force changes to UIs 
* **Patch**: bug fixes:
  * may change erroneous behavior but not "enhancement" bugs
 
## Changes in version 1.0.0 
_Perhaps this doesn't belong here, and will move somewhere else, but it's a topic of general interest and is related._

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
  * [{...en:n} Enable Setting](gpio-primitives#enn-enable-setting) used to enable and disable IO channels
  * [{...po:n} Polarity Setting](gpio-primitives#pon-polarity-setting) used to set polarity of IO channels
* Function (fn) has been removed from digital inputs:
  * [{di1ac: Action](gpio-primitives#digital-input-configuration-values) now handles all actions and functions
  * [{di1in: Exposed As](gpio-primitives#digital-input-configuration-values) allows flexible bindings of input readers to digital input channels

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
