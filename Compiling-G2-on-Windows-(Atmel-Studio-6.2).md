_This page is for compiling the G2 project on Windows with Atmel Studio 6.2. Please see [Getting Started with G2](Getting-Started-with-G2) for information about hardware and compiling on other platforms._

## What's needed

To compile G2 on Windows with Atmel Studio you will need the Atmel Studio 6.2 (build 1153) Installer – with .NET.

* Go to Atmel and download the [Atmel Studio 6.2 Installer – with .NET](http://www.atmel.com/tools/atmelstudio.aspx) install package.
  * Be sure to get he one **including the .NET part**. It's about 700 Mbytes.
  * Current build is build 1153.
  * They require you to either register or fill out a "guest" form. Otherwise it's free.
* Walk through the entire installation process. 
  * You will **not** need the Atmel Solutions framework when asked.
  * You **will** need the USB drivers when asked.

## Cloning the git repository.

The easiest way on Windows to clone the git repo is probably to use the GitHub Windows app.

1. Download and install the [GitHub app](https://windows.github.com/).
2. Log into the GitHub web site -- register if needed, it's free.
3. Browse to the [G2 project page](https://github.com/synthetos/g2) and then click on the `Clone in Desktop` button.
  * The GitHub application should open up, and ask where to save the new repository. The default location will probably be sufficient.
4. In the GitHub app, click on the unnamed menu in the top-left and then click on `edge` to checkout the edge branch.<br/>
![Choose edge from the unnamed menu near the top-left of the GitHub window](images/Windows-GitHub-Edge-Branch.png)
5. (Convenience) From the gear menu in the top-right of the Github window choose "Open in Explorer" to show the location of the newly checked-out repo.<br/>
![From the gear menu in the top-right of the Github window choose "Open in Explorer"](images/Windows-Github-Open-in-Explorer.png)

## Compiling

In the project directory all of the source files and the Atmel project files for both Studio 4 and Studio 6 are inside the `TinyG2` directory. Once Atmel Studio 6 is installed, open the solution file `tinyg.atsln`. (Atmel studio will also open the project file `tinyg.cproj` automatically.)

_Note:_ Git is configured to ignore the changes to some of the project's dependent files so that they don't cause havoc. This means that to commit changes to those files, they need to specifically be added to the commit by name.

To compile the project:
![](Windows-Choose-Build-And-Processor.png)

1. Choose the platform you are building for (for the Due with gShield pinout, choose `gShield`).
2. Click either the "Build Project" or "Build Solution" buttons -- they are the same in this case. (These can also be found in the Build menu.)
3. Make sure the correct processor is selected for programming with a Atmle ICE.
  * This step can be skipped if not using a debugger to upload the firmware to the board.
  * For the Due, it needs to read `ATSAM3X8E`, and for TinyG v9 is needs to read `ATSAM3X8C`.
4. (Optional) To upload _any_ binary (particularly useful if you didn't compile it), you can use the "Device Programming" tool (also available under the Tools menu) to upload to the board without compiling.
5. (Optional) To compile and upload without debugging (left) or with debugging (right) click one of these two buttons. These are also available from the Debug menu.

## Uploading G2 to a target board

### Atmel Studio6 Bugs, Workarounds, and Other Topics
Here are some known bugs that we have had to work around

#### Can't compile under -Os optimization
This seems to be a known bug with AVRGCC 4.8.1. The fix is to compile under -O2. Find this in the Properties (TinyG) / Toolchain / AVR/GCC Coplier / Optimization window. In the flags box that follows add: 
`-fno-align-functions  -fno-align-jumps  -fno-align-loops -fno-align-labels -fno-reorder-blocks -fno-reorder-blocks-and-partition -fno-prefetch-loop-arrays -fno-tree-vect-loop-version`

#### Can't view ASCII strings in the debugger
The problem is that ASCII display are shown as decimal or hedidecimal numbers. There is a way around this documented here: http://www.avrfreaks.net/index.php?name=PNphpBB2&file=printview&t=105137&start=0

* Add the variable as a quickwatch and append ",c" to it. If the var is part of a struct you need the entire reference, e.g. tg.inbuf,c  Then add the quickwatch to the watch. Of course this is impractical if you have a lot of ASCII vars to track. Once you have made these time-consuming changes you will want to save your project.

See also from the Studio6 Help Pages:

* _While debugging you might want to track a value of a variable or an expression. To do so you can right click at the expression under cursor and select Add a Watch or Quickwatch_
* _To add a QuickWatch expression to the Watch window_
* _In the QuickWatch dialog box, click Add Watch._

#### Floating point numbers appear as unit32's in the debugger

We don't have a workaround for this one yet. Perhaps something similar to the ASCII fix might work.

#### AtmelStudio6 fails verification when programming the xmega
We've seen this with earlier versions and Studio6.0 build 1996. We have not heard of this yet for AS6.2. Make sure you are up to Studio6 build 1996, service pack 2. See the `help` tab. Also make sure you are on the latest firmware for your programmer (Atmel-ICE or AVRISP mkii).

We've found that even though the verification failed the programming is good. You can either turn off verification or chose to ignore the warning. By the way - we've never seen this failure on Studio4. I actually think this is a bug in Studio6 rather than and actual failure. I have not seen any code die or act erratically when I get these errors.

#### Simulator won't start - Cannot find debugger message
Who knows why this happens. The get the debugger (simulator) back do the following
* Plug in and select an AVRISP mkII in the tinyg/tools tab
* Hit Debug and stop to try to debug. It will fail
* Go back to the tools tab and now select simulator for the tool
* Try to debug again. 60% of the time it works every time


(1) It gets worse. If you mess up and do things wrong studio6 will remember your paths in "recents" and fail the next time you try to set something up because things aren't where it thinks they should be. You have to go to the File/Recent Projects and Solutions, try to open the old paths, then remove them when they can't be found (you must have deleted the directories beforehand)