---
published: true
title: Configure Dell BIOS from WinPE
layout: post
---
To start off, I should clarify that I typically use SCCM Task Sequences with MDT integration.  The following scenario utilizes SCCM 2012 R2 with MDT 2013 integration.

1. Download and install the Dell Client Configuration Toolkit (CCTK) from Dell’s website: http://www.dell.com/support/home/us/en/04/Drivers/DriversDetails?driverId=6HCTN
2. From the installation directory, typically C:\Program Files (x86)\Dell\CCTK, grab X86 and X86_64 folders and create an empty package (no programs).

3. Additionally, add CCTK-default.cmd and show-cctkerrors.vbs to the root of your package folder, we’ll be utilizing this in a task sequence step.  These files were originally authored by windowsmasher: http://windowsmasher.wordpress.com/2013/01/21/sccm-2012-generic-multi-platform-dell-cctk-bios-settings/.  Thank you!  I’ve made my own customization to the cmd file to better suite my needs.  Created a config subfolder in the CCTK package and drop Multi-Platform_Generic.ini there.  You can find the files you need here.  The final folder structure should look like this: 
root\X86\
root\X86_64\
root\config\Multi-Platform_Generic.ini
4. Edit your OSD Task Sequence and insert a new folder Configure Dell BIOS.  I prefer to insert this at the Preinstall phase of the task sequence, although it technically can be applied anytime before the Apply Operating System image step in the task sequence.  This is because we reconfigure the SATA operating mode to the highest level that the BIOS is capable of, RAID On, AHCI, ATA, respectively.

5. You must also remember to create evaluations in the options of this new folder to ensure that the PC is actually running in WinPE and that the Manufacturer is Dell.

6. The next step is optional, but recommended for easily troubleshooting BIOS configuration issues.  Create a step called Connect to Network Drive.  This drive will be used for dumping log files in a subsequent step.

7. After optionally mapping a network drive, it’s time to perform the actual configuration.  It’s recommended that you read through the cmd file and make appropriate modifications to best suite your environment.

8. We must restart the computer to have the BIOS settings take effect.  This is particularly important for the SATA operation to ensure that the appropriate storage drivers are installed.  Make sure you reboot to WinPE, there is no operating system installed yet.

9. After rebooting, we need to re-load the toolkit package and run the gather step again.

