---
published: true
title: Custom log size in WinPE during Task Sequences
layout: post
---
Do you hate doing things manually?  Do you hate reading through forums and blogs to solve problems?  I do, that’s why I write scripts to solve problems.  They are concise and usually fairly readable.  I referenced Niall Brady’s blog: http://www.windows-noob.com/forums/index.php?/topic/11071-how-can-i-increase-the-smstslog-file-size-for-pxe-based-os-deployments-using-system-center-2012-r2-configuration-manager/  for the solution, so thank you very much Niall!  I have difficulty remembering the individual steps at times, so I like to script it out as a quick reference and to create a repeatable process.  In this case, I wanted to increase the log size while in WinPE during Operating System Deployment (OSD) task sequences.  If you want the specific detail, please refer to Niall’s blog, otherwise modify the variables in the script to suit the needs of your environment.  Just change the variables, run the script, and update your boot images on the DPs and you’re all set!

Here’s a link to the script: http://1drv.ms/1U58mzr