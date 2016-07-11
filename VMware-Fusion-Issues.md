This page discusses some VMware Fusion issues we've had and some fixes while working on this project

### Windows 10 and VMware 8.1.1 and OSX 10.10.5
Bad combination. I'm not sure who's at fault, but it looks like VMware. The Z drive kept disappearing and screen text magnifications and other settings never stuck. 

Symptom: The Z: drive that VMware is supposed to give you (\\vmware-host\Shared Folders) was broken with a big red X. The desktop shortcut to the same files, however, worked

Solution: Root cause after many hours of failed attempts: With VMware fusion 8.1.1 when you install vmware tools if you accept the "Restart" option (YES) it discards any changes. WHAT? 

The most reliable way to re-up vmware tools is to completely uninstall and re-install it with manual shutdowns. 
- Reinstall vmware tools. You must use the Reinstall VMware Tools option in the VMware Virtual Machine tab as VMware tools as not accessible from within the VM itself. Click on the D: Drive dialog bar to select the Setup action. Select Remove VMware tools. Do not accept the Restart option when complete.
- Shut down (completely). Reboot. 
- Reinstall again. Do a COMPLETE install from the Vmware tools installation menu. Do not accept the "Restart" menu option. Do a full shutdown and restart on your own. 

Big bug on EMC's part?
