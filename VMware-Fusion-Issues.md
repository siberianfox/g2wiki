This page discusses some VMware Fusion issues we've had and some fixes while working on this project

### Windows 10 and VMware 8.1.1 and OSX 10.10.5
Bad combination. I'm not sure who's at fault, but it looks like VMware. The Z drive kept disappearing and screen text magnifications and other settings never stuck. 

Symptom: The Z: drive that VMware is supposed to give you (\\vmware-host\Shared Folders) was broken with a big red X. The desktop shortcut to the same files, however, worked

Solution: Root cause after many hours of failed attempts: With VMware fusion 8.1.1 when you install vmware tools if you accept the "Restart" option (YES) it discards any changes. WHAT? 

The most reliable way to re-up vmware tools is to completely uninstall and re-install it with manual shutdowns. 
- Be sure Sharing is enabled in the VMware Settings for this VM
- Use `Reinstall Vmware Tools` to remove vmware tools completely. You must use the Reinstall VMware Tools option in the VMware Virtual Machine tab as VMware tools as not accessible from within the VM itself. Click on the D: Drive dialog bar to select the Setup action. Select Remove VMware tools. Do not accept the Restart option when complete.
- Shut down. Reboot.
- Now `Install VMware Tools`. Repeat the above steps by select `Complete` from the Vmware tools installation menu (`Typical` may also work). Do not accept the "Restart" menu option. Verify that you have proper screen resolution, and that the Z drive appears in the This Computer menu. Do a full shutdown and restart on your own. 
- On reboot you may be presented with a dialog about VMware tools having changed settings for sharing. Don't accept the reboot option. Instead, do the same manual shutdown and restart as above. You can verify that the Z drive is present beforehand, and once you have rebooted.

Additional optional steps

- Run Disk Cleanup once things are set up. Use the `Clean up system files` to find the old Windows 7 directories, and the Windows 10 temporary installation files. You can shave as much as 20 Gb off the size of the VM by deleting these.


