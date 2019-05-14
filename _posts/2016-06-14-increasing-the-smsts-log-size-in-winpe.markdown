---
published: true
title: Increasing the smsts.log size in WinPE
layout: post
---
If you're reading this, than like our organization, you've lost some log history in the smsts.log file that may have been important for troubleshooting an issue.  I started seeing this problem after we integrated MDT into our task sequences.  The capabilities MDT gives are you are really nice, however it does come with some downsides, such as significantly longer task sequences.  And by longer, I mean many, many more steps.  With more steps comes more logging in the smsts.log file.  Unfortunately, you may have noticed that this file rolls over at approximately 1MB in size, appends a time and date stamp for a 'log history file,' then starts a new file.  This can be problematic searching for text in multiple files and can be very frustrating when losing log history altogether.  MDT task sequences have many steps right out of the box, you are more than likely going to lose some of that log history if you don't make some changes.

This problem can be resolved by creating a smsts.ini file in the WinPE image to specify a larger log size and how many log history files are retained.  Doing this consistently, especially when you need to to rebuild the boot images on occasion can be quite time consuming and error-prone.  I wanted to have some consistency and an easy way to change the log size without having to think about it too much, so I wrote this PowerShell script to do the work for me.  I wrote them in a few functions as I had intended to fully test them with Pester (and still hope to), but in the meantime, the file can be found here: <a href="https://github.com/dansonnenburg/Add-SmstsIniToBootImage">Visit my GitHub project</a>

To execute, load the functions into memory, then execute: <code>Set-SMSTSLogSizeInWinPE -$WinPEWim <PathToYourBootWIMFile></code>

By default, the script increases the log size to 5MB and increases the maximum log history to 3.