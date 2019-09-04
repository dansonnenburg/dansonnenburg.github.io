---
published: true
title: Backup and Restore a WIM from WinPE using DISM
layout: post
---
Backup an image
1. Boot from WinPE media.
2. Determine the drive letter to be captured.  This is typically C:,D:, or E:, but it can vary depending on which devices are connected to the system.
3. There are a few choices for where to back up the image, but my preference is to back it up to a network drive.  To do this, run the following command: 
Net use Z: \\\\
You should see “The command completed successfully” if it works correctly.  Remember, you need to have write permissions to this directory in order to save the image to it, so you should be prompted for credentials to connect to the share.
4. DISM.exe should already be in the path environment variable in WinPE 5.0, so you can call the DISM executable from pretty much anywhere.
DISM.exe /Capture-Image /ImageFile:Z:\capture.wim /CaptureDir:E:\
Where Z:is the path to which you want to save the image and E: is the path to the partition with Windows installed.
If the drive contains much data, it will take an amount of time proportional the amount of data and the throughput of your network, USB, or eSATA connection.  Be patient
Restore an image

To restore an image to a new machine or a new drive, it is a very similar process, only the command in step 4 really changes.
1. Boot from WinPE media.
2. Determine the drive letter to restore the image.  This is typically C:,D:, or E:, but it can vary depending on which devices are connected to the system.
3. If you used a network path to backup your image, you must re-map the drive again.  To do this, run the following command.
<code>Net use Z: \\\\ </code>
You should see “The command completed successfully” if it works correctly.  Remember, you need to have write permissions to this directory in order to save the image to it, so you should be prompted for credentials to connect to the share.
4. DISM.exe should already be in the path environment variable in WinPE 5.0, so you can call the DISM executable from pretty much anywhere.
<code>DISM.exe /Apply-Image /ImageFile:Z:\install.wim /Index:1 /ApplyDir:E:\ </code>
Where Z:is the path where the backup image is stored and E: is the path of the partition to restore the image.
 If the image contains much data, it will take an amount of time proportional the amount of data and the throughput of your network, USB, or eSATA connection.  Be patient!