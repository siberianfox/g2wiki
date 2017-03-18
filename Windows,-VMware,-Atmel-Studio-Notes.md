This is a notes page for things I found when installing and using the Windows dev environment. Presumably relevant portions of this page will be extracted and turned into real project documentation later.

## Atmel Studio Migration
For some period of time we will have some branches that were maintained using Atmel Studio 6.2, and others that have been migrated to Atmel Studio 7. Atmel Studio has problems with the following files because they are part of the local environment and not part of the git repo that gets moves in and out:
- TinyG2.atsln - solution file - should not need to be changed
- TinyG2.cppproj - project file - specific to AS6 or AS7 and incompatible if corossed
- TinyG2_6_2.cppproj - project file occurring to save AS6.2 project data during AS7 migration
- TinyG2.atsuo - local environment file - specific to AS6 or AS7 (and incompatible if crossed)
- Motate.cppproj - similar to above
- Motate_6_2.cppproj - similar to above

### Migrating an AS6 branch to AS7

### Changing from an AS6 branch to an AS7 branch

### Changing from an AS7 branch to an AS6 branch

- If you switch your git checkout from an AS7 branch to an AS6 branch you need to do the following for AS6 to load and compile:
  - Delete the .atsuo files in both the TinyG directory
  - Rename Motate.cppproj to Motate_AS7.cppproj. Copy and rename Motate_6_2.cppproj to Motate.cppproj
  - Run git submodule update. It should replace Motate w/one that works for that branch
