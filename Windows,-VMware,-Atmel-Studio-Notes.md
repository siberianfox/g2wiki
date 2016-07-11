This is a notes page for things I found when installing and using the Windows dev environment. Presumably relevant portions of this page will be extracted and turned into real project documentation later.

##Atmel Studio Migration
For some period of time we will have some branches that were maintained using Atmel Studio 6.2, and others that have been migrated to Atmel Studio 7.

###Migrating an AS6 branch to AS7

###Changing from an AS6 branch to an AS7 branch 

###Changing from an AS7 branch to an AS6 branch 
 
- If you switch to AS6 from a branch that was in AS7 you need to do the following for it to load and compile:
  - Delete the .atsuo files in both the TinyG directory
  - Rename Motate.cppproj to Motate_AS7.cppproj. Copy and rename Motate_6_2.cppproj to Motate.cppproj
  - Run git submodule update. It should replace Motate w/one that works for that branch

