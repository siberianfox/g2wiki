# --- DRAFT ---
_**This page discusses proposed changes to the branching and release scheme for g2core. Please feel free to comment and edit.**_

# Current Branch and Release
* **Master branch** - is currently the stable release branch. Pushes to master are performed periodically as code gets sufficient field history from early adopters of edge code
* **Edge branch** - is currently the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* **Issue-NNN branches** - are created to work on bugs or features. These are pulled into edge via pull requests. 
* **Dev branches** - The Dev prefix has already been deprecated, but you may still see some dev branches around.

# New Branch and Release
The proposed scheme is below:
* **Master branch** - is the working branch. Bug and feature updates are performed against this branch. All pull requests should be performed from this branch.
* ** Release branches ** are branches for creating releases. These are created from master when a Major or Minor release is planned. Subsequent development occurs again the release branch, and is usually limited to version number changes and tagging. The release branch  is used to create binaries for the release section, put out release notes, update the wiki with info about it, etc. Bug fixes that come in against the release are addressed as issues/PRs in master, then cherry picked back to the release branch. After a certain point the version may be updated with a new point release (patch) and new binaries created.
* **Issue-NNN branches** - are created to work on bugs or features. These are pulled into Master via pull requests against Master.
* Deprecated and Older Branches
  * Edge branch - Edge will be deprecated and ultimately removed. 
  * Dev branches - The Dev prefix has been deprecated, but you may still see some dev branches for a while
  * Legacy branches such as digital-dro and omc-072.65 may still exist for historical reasons

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
 

