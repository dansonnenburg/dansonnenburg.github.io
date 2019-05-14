---
published: true
title: Invoke MBAM in Windows 10 v1607 and higher
layout: post
categories: [MBAM, ConfigurationManager]
---
We recently discovered that TPM ownership had not been taken during OSD on a majority of our devices.  In the past, all we had to do was run two steps in the 'State Restore' section of our Windows 10 task sequence.  First, install the MBAM agent, then start MBAM encryption via the <code>Invoke-MbamClientDeployment.ps1</code> script.

From the logs, we were able to see the Invoke-MbamClientDeployment.ps1 script return with the exit code 1: 'MBAM cannot take the ownership of TPM because auto-provisioning is pending. Try again after the auto-provisioning is completed. Throw "Failed to prepare TPM for encryption."'

After much troubleshooting, I determined that the TPM ownership was not occuring during OSD.  In versions prior to Windows 10 v1607, MBAM was able to take ownership of the TPM directly, for v1607 and higher only Windows is able to take ownership of the TPM, by design. (Source: https://docs.microsoft.com/en-us/microsoft-desktop-optimization-pack/mbam-v25/how-to-enable-bitlocker-by-using-mbam-as-part-of-a-windows-deploymentmbam-25)

To remedy this situation, I just had to add a step to initialize the tpm before running the invoke.  This is accomplished with a simple Powershell command that has resolved all of our problems, as seen below:
<code>Powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Initialize-TPM"</code>

The invoke step looks like the following: <code>Powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\Invoke-MbamClientDeployment.ps1 -RecoveryServiceEndpoint http://ServerName/MBAMRecoveryAndHardwareService/CoreService.svc -EncryptionMethod AES256 -IgnoreEscrowRecoveryKeyFailure -IgnoreEscrowOwnerAuthFailure</code>

I hope this helps someone else avoid some of the challenges we were having with MBAM and TPM in OSD.  Good luck!