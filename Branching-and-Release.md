# --- DRAFT ---
_**This page discusses proposed changes to the branching and release scheme for g2core. Please feel free to comment and edit.**_

# Current Branch and Release
* Master branch - is the stable release branch. Pushes to master are performed periodically as code gets sufficient field history from early adopters of edge code
* Edge branch - is the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* Issue-NNN branches - are created to work on bugs or features. These are pulled into edge via pull requests. 
* Dev branches - The Dev prefix has been deprecated, but you may still see some dev branches around.

# New Branch and Release
The proposed scheme is below:
* Release branch - is the stable release branch. Pushes to Release are performed periodically as code gets sufficient field history from early adopters of edge code. Some patches may be applied to Release, but these should always be promoted up through Master. New feature development should not be done against Release. 
* Master branch - is the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* Issue-NNN branches - are created to work on bugs or features. These are pulled into Master via pull requests against Master. 
* Edge branch - this branch will be deprecated and ultimately removed. 
* Dev branches - The Dev prefix has been deprecated, but you may still see some dev branches around.

## Semver and Changes to ID attributes
The new branches will work to a fairly standard semantic versioning scheme - major.minor.patch:
* Major: Anything can change, including:
  * changes to machining behaviors
  * changes that affect the semantics of existing JSON variables
  * changes that require configuration files to revised
    * e.g. polarity sense
  * minor or patch level changes may also be included, but do not force a Major revision
* Minor: Limited things can change, including:
  * new machining or other features
  * new JSON objects can be added
  * new JSON tags can be added to existing objects
    * ...as long as they do not change the semantics of existing attributes
  * new values to existing objects
  * patch level changes may also be included, but do not force a Minor revision 
* Patch: bug fixes:
  * may change erroneous behavior but not "enhancement" bugs
 
do we mean by:
Major.minor.

### ID changes
Proposed changes to IDs:
* {fv:Major.Minor.Patch} - is now a string, not a float
* {fb:ShortSha} - is now a string, not a float

The following remain the same:
* {fbc:Settings file}
* {hp: board string - e.g. gQuintic
* {hv: becomes a board string - e.g. revF
* {id: remains the same
