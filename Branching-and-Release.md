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
* **Patch**: bug fixes:
  * may change erroneous behavior but not "enhancement" bugs
 
## Changes in version 1.0.0 
_Perhaps this doesn't belong here, and will move somewhere else, but it's a topic of general interest and is related._

Once the new branching and versioning is in place a merged branch labeled `1.0.0` will be pushed. This combines the work that has been done on edge (the ARM M3 code base) with the work done in the ARM M7 code base. They will then be a single code base with the ARM chip being selected by the board hardware.

This section lists the changes that may be incompatible with previous releases (either branch). This is an attempt to get these all together for the 1.0.0 release and take the hit at one time.

### Changes to ID Attributes
Proposed changes to IDs:
* {fv:Major.Minor.Patch} - is now a string, not a float
* {fb:ShortSha} - is now a string, not a float

The following remain the same:
* {fbc: settings file - e.g. settings_axidraw_v3.h
* {hp: board platform string - e.g. gQuintic
* {hv: board version string - e.g. revF
* {id: board ID string - e.g. 0271-6fd3-e29c-14d

### Changes to Polarities
All polarities now obey 0=ACTIVE_HI (non-inverted polarity), 1=ACTIVE_LO (inverted polarity). This changes: 
* Coolant
  * Coolant mist polarity        // {comp:} 0=active HI, 1=active LO
  * Coolant flood polarity       // {cofp:} 0=active HI, 1=active LO
* Spindle
  * Spindle enable polarity      // {spep:} 0=active HI, 1=active LO
  * Spindle direction polarity   // {spdp:} 0=clockwise HI, 1=clockwise LO

The following remain the same:
* Motor direction polarity {1po:n}
* Motor enable polarity {1ep:n}
* Motor step polarity {1ep:n}
