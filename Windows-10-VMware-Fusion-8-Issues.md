We use Atmel Studio 7 on Windows 10 from OSX for a lot of g2 development. This page discusses some VMware Fusion issues we've had and some fixes trying to get an upgraded Windows 7 --> Windows 10 to work correctly on VMware. 

Our approach has been to build a fresh Windows 7 install from DVD, Skip installing W7 updates (not necessary and takes many hours), go to MS and request the free Windows 10 upgrade (pre-July29, 2016).

Revisions Used:
- Windows 10 downloaded early July, 2016
- VMware 8.1.1
- OSX 10.10.5
- Atmel Studio 7 - 1009

### Showstopper Bug - VMware Tools Installation Failures
These problems make it impossible to use Atmel Studio 7.

**Symptom**: Screen resolution cannot be set above 1440x920, 1600x1200 or in some cases 1143x876 (can't remember the exact setting). This is the so-called `svga bug` which has been fixed, but only if VMware tools is successfully installed.

**Symptom**: The Z: drive that the `This Computer` display is supposed to give you (\\vmware-host\Shared Folders) appears, but is broken with a big red X. The `Network` Display does not show `vmware-host`  and `Shared Folders` like it's supposed to. The desktop shortcut to the same files, however, works. If the desktop shortcut does not work you may have other problems such as:
- Sharing was not enabled from within VMware
- No or bad credentials for the host operating system (try connect with different credentials)
- Windows firewall rules prevent connection (but the default ON settings should work)

**Solution**: I think the root cause is that VMware tools installations do not "stick if you accept the `Restart` option at the end of the Vmware tools installation dialog. If you accept the `Restart` option (YES) it seems to discard any changes. WHAT?

The most reliable way I've found to re-install vmware tools is to completely uninstall and re-install it with manual shutdowns and restarts after each step. (`Restart` from the Windows `Power` button may also work - I'm not sure).
- Be sure Sharing is enabled in the VMware Settings for this VM
- Invoke `Reinstall Vmware Tools` from the VMware Virtual Machine tab to remove vmware tools completely. (VMware tools installation is not accessible from within the VM itself). Click on the `D: Drive` dialog bar to select the `Setup` action or the installation will not run (Autorun appears to be broken). Select `Remove` VMware tools. Do not accept the `Restart` option when complete.
- Shut down from the Windows `Power` display. Reboot the VM.
- Now `Install VMware Tools`. Repeat the above steps by select `Complete` from the Vmware tools installation menu (`Typical` may also work - I'm not sure). As above, do not accept the `Restart` menu option. Use the `This Computer` display to verify that the Z drive appears w/o a red X, and that you have proper screen resolution. Do a full shutdown and reboot manually. 
- On reboot you may be presented with a dialog about VMware tools having changed settings for sharing. Don't accept the `Restart` option. Instead, do the same manual shutdown and reboot as above. You can verify that the Z drive is present beforehand, and once you have rebooted.

Additional optional steps once this works:
- Run `Windows Update` and install new ones, then...
- Run `Disk Cleanup` once things are set up. Use the `Clean up system files` to find the old Windows 7 directories, and the Windows 10 temporary installation files. You can shave as much as 20 Gb off the size of the VM by deleting these. The VM should drop to about 11 to 14 Gb. You will not be able to restore to W7 on this VM if you do this.

### Annoyance Bugs
These problems are annoyances and can be worked around.

- `Custom Scaling` settings cannot be changed to reduce the size of Windows screen objects. Attempt: Turn off `Custom Scaling` by right clicking the main display and selecting `Turn Off Custom Scaling and Logout`. Once logged in again VMware tools wants a reset. Cancel this and perform the shutdown/restart manually. The you can select you own text scaling factor from the `Display Settings` dialog. Sometimes this doesn't even work. Display settings still revert.

- Mouse cursor gets locked into VM single-window screen. Unlock with Control - Command (cloverleaf) or change windows with Command-Tab window navigator

- Some VM's will occasionally "beachball" forever and need to be killed from above.

- Many dialog windows pop under existing windows instead of on top. This might be a W10 problem. But if you get stuck be sure to look under windows to see if there;'s something you are supposed to click.

### Tips
- When starting a new VM image from a W7 disk, name the VM as W10, as you will be upgrading to W10. The internal package files are named after the parent VM name. I'm not sure if this can be changed later. This is not critical - just a clarification.

- Once you have the OS where you want it - VMware tools installed, disk cleaned, etc., but before you start installing additional software, make a BASELINE snapshot for recovery purposes.

- Some OEM W7 disks will not activate for W10 installation. You will find this out after you have spent the time installing W7 from that disk.
