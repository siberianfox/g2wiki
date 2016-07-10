This page discusses some VMware Fusion issues we've had and some fixes while working on this project

### Windows 10 and VMware 8.1.1 and OSX 10.10.5
Bad combination. I'm not sure who's at fault, but it looks like VMware. The Z drive kept disappearing and screen text magnification never stuck. 

Root cause after many hours of failed attempts:

With VMware fusion 8.1.1 when you install vmware tools if you accept the "reboot now" option it discards any changes. WHAT?The most reliable way to re-up vmware tools is to completely uninstall and re-install it. Reboot. Then do a COMPLETE install from the vmware tools installation menu. Then do not accept the "reboot now" option. Instead either use it, or do a full shutdown on your own. Big bug on EMC's part?
