---
published: false
title: Lessons learned from Windows 10 Upgrade task sequence.
layout: post
---
You must remove App-V 4.6 SP1 from the currently installed operating system before running the Windows 10 Upgrade task sequence.  App-V is incompatible with Windows 10 and prevents non-admin accounts from logging into the workstation following the upgrade.