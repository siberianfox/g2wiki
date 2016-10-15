# Prerequisites

### GNU build system

Download and install _**MinGW**_, installation is covered here: [Getting Started](http://www.mingw.org/wiki/getting_started)  
Cygwin is another option to using MinGW, but is not currently covered here.  
This provides you with the GNU build system that is required to compile many cross platform and embedded open source projects.

### GNU tools for ARM

You also need a copy of _**GNU tools for ARM embedded**_, which can be obtained here: [GNU ARM Embedded](https://launchpad.net/gcc-arm-embedded/+download)  
This provides you with the C and C++ cross compilers required to compile ARM code on your Windows machine.  
I strongly recommend you install this a low-level folder **without spaces** to avoid problems.

**Note:** I will be using _C:\GTAE54\_ for all further illustrations.

# Getting the source

If you are familiar with Git, simply clone the repository into a local folder - but remember there are submodules.

If you do not want to use Git, you can just download any repository as a .zip file:

Go to the [G2 repository](https://github.com/synthetos/g2) and click _Clone or download_, then _Download ZIP_ and unpack that file to a local folder. Click the **_Motate submodule_** and repeat the process. The Motate folder should go in the G2 folder such that your local folder content matches that of the G2 repository.

**Note:** I will be using _C:\cnc\g2\_ for all further illustrations. You may use any folder you like.

# Telling Motate about your ARM toolchain

Looking in _C:\cnc\g2\Motate\Tools\win32_, the folder should be empty at this time.

You need to create a folder here called _gcc-arm-none-eabi_ which contains the ARM toolchain to use when compiling G2.
There are two ways to do this: _symlink_ or a simple _copy_.

As for the _copy_ method, simply copy your _GNU tools for ARM embedded_ folder to _C:\cnc\g2\Motate\Tools\win32_ and name it _gcc-arm-none-eabi_.

The _symlink_ method involves creating a link that looks like a folder but actually just references another folder.
Start an administrator command prompt and navigate to _C:\cnc\g2\Motate\Tools\win32_ and run ```mklink /D gcc-arm-none-eabi C:\GTAE54```

# Compiling

Open a command prompt and navigate to _c:\cnc\g2\g2core\_.

Run ```make CONFIG=Othermill```

You should now have a binary at _c:\cnc\g2\g2core\bin\Othermill-g2v9k\g2core.bin_

### Troubleshooting

    'make' is not recognized as an internal or external command,
    operable program or batch file.

There is a problem with your MinGW installation, or your PATH environment variable is not properly set up. Please consult the MinGW getting started article again.

    ... && make "ARCH=gcc-arm-none-eabi"
    /usr/bin/bash: make: command not found

You have not set up your symlink/copy of the ARM toolchain, please refer to the previous chapter.

# Available configurations

Open c:\cnc\g2\g2core\boards.mk in your favorite text editor.

Here you will find a list of all available configurations for use with the ```make CONFIG=``` command.

# Customization

### Custom settings

_When you have a standard machine and just want to tweak things a bit._

Open _c:\cnc\g2\g2core\settings\settings-\<YourMachine\>.h_ in your favorite text editor and change the settings.

After you make your changes, just re-compile and upload to the board to test it out.

### Custom configuration

_When you have connected a custom built or heavily modified machine to a supported board._

Open _c:\cnc\g2\g2core\boards.mk_.

Looking at the existing configurations, choose a board to use as a template and make a copy of it. Right underneath is fine.  
Name your configuration by editing this line: ```ifeq ("$(CONFIG)","MyNewMachine")```  
Change the settings file by editing this line: ```SETTINGS_FILE="settings_MyNewMachine.h"```

You may now make a copy of the original settings file in _c:\cnc\g2\g2core\settings\_ and modify that according to your hearts desires.

When it is time to compile, you can now ```make CONFIG=MyNewMachine```

### Custom Board

_When you have have built your own custom controller or shield._

Change the board name by editing this line: ```BOARD=MyCustomBoard```

Open _c:\cnc\g2\g2core\board\ArduinoDue.mk_.

Looking at the existing boards, choose your template and make a copy of it.

Name your board by: ```ifeq ("$(BOARD)","MyCustomBoard")```  
Change pinout file by: ```DEVICE_DEFINES += MOTATE_BOARD="MyCustomBoard"```

You may now make a copy of the original pinout file in_c:\cnc\g2\g2core\board\ArduinoDue_ and modify that according to your custom board.
