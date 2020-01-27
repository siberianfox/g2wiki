# Setup of VS Code

**NOTE: This is a draft, for an unsupported branch that’s not fully tested and released yet.**

## Aside: Why VSCode

Microsoft’s [Visual Studio Code](https://code.visualstudio.com) (VSCode for short), *not to be confused with Microsoft’s older (and still maintained) Visual Studio*, is a complete language-agnostic Integrated Development Environment (IDE) that is cross-platform (Mac, Windows, and Linux), free, open-source, and extensible.

Compared to other IDEs, VSCode had the best mix of capabilities for our unique set of requirements for the g2core project. We needed it first and foremost to support bare-metal embedded code-editing, compiling, and debugging. We also had a strong desire for it to be cross-platform to support development without needing a Windows VM on Macs or on Linux-based computers. VSCode gets extra points for having great support for [Windows Subsystem for Linux (WSL)](https://code.visualstudio.com/docs/cpp/config-wsl) to ease development on Windows without having to maintain a separate set of configurations for Windows.

## References

- [VSCode Download Page](https://code.visualstudio.com)
- [VSCode C++ Extension Documentation](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [VSCode C++ FAQ](https://code.visualstudio.com/docs/cpp/faq-cpp)

# *TODO*

- Document Window differences
- Document Linux differences

# Precursor Steps

These instructions assume you have already cloned the g2 repo locally using another means. The g2 repo should contain the g2core subdirectory, and a Motate subdirectory, among other things.

Github has advanced [token management](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) that may be helpful.

# Installation

1. Install [VSCode](https://code.visualstudio.com)
   * See their [setup documentation](https://code.visualstudio.com/docs/setup/setup-overview) 
   * If you’re new to VSCode, then their [introductory videos](https://code.visualstudio.com/docs/getstarted/introvideos) may be of interest.
2. Follow the setup instructions for your platform: [Mac](https://code.visualstudio.com/docs/setup/mac), [Linux](https://code.visualstudio.com/docs/setup/linux), [Windows](https://code.visualstudio.com/docs/setup/windows)
   * Note, for Windows we recommend using the [Windows Subsystem for Linux (WSL)](https://code.visualstudio.com/docs/cpp/config-wsl) to get the development tools
3. Open VSCode, and then open the g2core repo
4. Install the [recommended extensions](https://code.visualstudio.com/docs/editor/extension-gallery#_recommended-extensions) (left hand nav bar "blocky" symbol - see [documentation](https://code.visualstudio.com/docs/editor/extension-gallery#_browse-for-extensions))
   * [VSCode Extension usage documentation](https://code.visualstudio.com/docs/editor/extension-gallery) 
   * Note that you can optionally open the Extensions sidebar and browse the extensions that are currently active. You can safely disable any unused extensions *for this workspace* to speed up VSCode when working in g2core. Any extensions **not** related to C++, “Cortex-Debug,” Git, Github, WSL (on Windows), or IntelliCode can safely be disabled *for this workspace*.

**Usage Note**: Unlike other IDEs which use most of the screen real-estate for buttons, each for a single task, VSCode (much like [Atom](https://atom.io)) uses the type-to-find-commands approach, and they call it the [Command Palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette) (`<COMMAND><SHIFT>p`).

Some functionality can *only* be found via the Command Palette, so it’s important to learn how to use it. Note that all of the functions found there either already have keyboard shortcuts (shown in the list when you search) or can be assigned [keyboard shortcuts](https://code.visualstudio.com/docs/getstarted/keybindings) (note you can assign individual [tasks](https://code.visualstudio.com/docs/editor/tasks#_binding-keyboard-shortcuts-to-tasks) their own shortcuts as well).

Also note: As a general rule we try not to add keyboard shortcut configuration to the committed configuration files in the repo, as it may conflict with personal preferences or setup for other projects.

## Also needed

You will need the following command line tools to be installed for compiling:

* `git`  . Install GitHub Desktop: https://desktop.github.com or https://git-scm.com/download
* `node` . Install the LTS version: https://nodejs.org/en/
* **Others? TBD**
* [J-Link Software and Documentation Pack](https://www.segger.com/downloads/jlink#J-LinkSoftwareAndDocumentationPack) for flashing and debugging

### Hardware needed

* A J-Link hardware debugger
  * Adafruit sells two options which are much more reasonably priced for hobbiest uses:
    * [J-Link EDU](https://www.adafruit.com/product/1369), you’ll also need an [adapter](https://www.adafruit.com/product/2094) and a [cable](https://www.adafruit.com/product/1675)
    * [J-Link EDU Mini](https://www.adafruit.com/product/3571), you may also want to grab an extra [cable](https://www.adafruit.com/product/1675), but the Mini comes with one
* A compatible g2core device, of course 😉 

# Building

**TODO**

Notes:
- The workspace already has [Tasks](https://code.visualstudio.com/docs/editor/tasks) configured for building and cleaning the repo for various targets.

Here are some step-by-step instructions to use this (which will probably require edits)
- Open a terminal window using Terminal / Open Terminal
- Ensure Motate is up to date: `git submodule update`
- Type `tasks` in Command Palette

Briefly: Under the `Terminal` menu (or using the [Command Palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette)) choose `Run Build Task`. (Note: The keyboard shortcut for your OS will be shown next to `Run Build Task`, note it for faster access in the future.)

# Debugging

**TODO**

We have already configured Debugging for several targets, all using the J-Link tools. Follow the VSCode [documentation](https://code.visualstudio.com/docs/editor/debugging) (additional notes for [C++ here](https://code.visualstudio.com/docs/cpp/cpp-debug)), using one of those profiles.

# Notes (To be incorporated above and then deleted)

Windows:

* Install gitHub Desktop
* Install VSCode
  * Install Recommended Extensions
    * May be warnings about things already installed and not being updated - safe to ignore
* Install WSL (https://docs.microsoft.com/en-us/windows/wsl/install-win10) and The in the Windows Store install Ubuntu
  * Note the command to enable WSL needs to run in a PowerShell running as Administrator. Search for PowerShell and click on “Run as Administrator” and then wave away all the various warnings about the dangers.
* Connect to Ubuntu
* Open Cloned Repo (may be easier if it’s cloned inside Ubuntu)
* In the shell, run `sudo apt-get install build-essential`

