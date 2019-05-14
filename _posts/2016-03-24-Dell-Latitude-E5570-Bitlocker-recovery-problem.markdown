---
published: true
title: Dell Latitude E5570 Bitlocker recovery problem
layout: post
---
We recently discovered a problem with Bitlocker on the Dell Latitude E5570 laptops, that after enabling bitlocker (we use MBAM), the computer prompts for a recovery key after every reboot.  It turns out this is a problem with the Dell BIOS which is repaired through a BIOS update, though it has been noted that running in UEFI mode may fix this problem as well.  There is a nice discussion in this issue here: <a href="http://en.community.dell.com/support-forums/laptop/f/3518/t/19674757?pi41097=1"></a>

The root-cause of this problem is an issue with the default BIOS version on E5570, E5470, and E7270s.  Dell released a BIOS fix for this on March 16th.
To repair, do the following:

1.      Be sure that you are plugged into a power source.
2.      Install the BIOS update, located here: <a href="http://www.dell.com/support/home/us/en/04/product-support/product/latitude-e5570-laptop/drivers/advance"></a>
3.     After reboot you will be prompted to suspend and resume bitlocker, press enter.
4.     Enter the bitlocker recovery key using the bitlocker help desk portal.
5.     From an elevated command prompt type:
<code>Manage-bde.exe –protectors –disable c:
Manage-bde.exe –protectors –enable c: </code>


6.     Reboot the computer to ensure that the problem has been resolved.