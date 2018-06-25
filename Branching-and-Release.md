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
Things to record and make reportable, and as long as we are in the code making ID changes:

* {ver:Major.Minor.Patch}
  * deprecate {fv:
  * alternately we could take over fv with a string: {fv:maj.min.pat}
* {sha:ShortSHA}
  * deprecate {fb:
  * alternately we could take over fb with a string: {fb:shortSHA}
* {fbc:Settings file}
* {hp: become a board string - e.g. gQuintic
* {hv: becomes a board string - e.g. revF
* {id: remains the same
* key: 6 digit code to access cloud-stored records from the board, including its test record
  * built as modulus of salted hash of ID
  * key is not actually on the board
  * just spitballing here