---
published: true
title: Missing MBAM WMI classes on client computers
layout: post
categories: [MBAM, ConfigurationManager]
---
First, a bit of background: We had MBAM 2.5 SP1 with Configuration Manager 2012 R2 integration...or at least we did originally.  We later upgraded to Configuration Manager 1511, then again to 1602.

Every time that I think I understand MBAM, it seems to throw a new curve ball at me.  This time, the curve ball was missing <em>Win32_BitlockerEncryptionDetails</em> and <em>Win32Reg_MBAMPolicy</em> WMI classes.  It was my belief that the MBAM 2.5 SP1 client installation compiled both of the aforementioned MOF files on the local machine, but low and behold, all of our new computers were missing these particular WMI classes.  An easy way to check for the presence of these classes is by running the following PowerShell commands.  If errors are returned, then the classes are not present.

<code>gwmi -query "select * from Win32_BitlockerEncryptionDetails"<br>
gwmi -query "select * from Win32Reg_MBAMPolicy"</code>


As it turns out, the part that I misunderstood (or forgot about) was that the missing WMI classes for MBAM were added to the <em>configuration.mof</em> file found in: <em>"\\\\<ServerName\>\C$\Program Files\Microsoft System Center 2012\Configuration Manager\inboxes\clifiles.src\hinv\configuration.mof"</em> and were lost after our Configuration Manager upgrade.  The resolution is pretty straight-forward after determining this was the root of my problem.  All I needed to do was copy the following block to the end of the <em>Configuration.mof</em> file and let Configuration Manager distribute the file and compile the MOF on each individual system.  Problem solved!

```
//===================================================
// Microsoft BitLocker Administration and Monitoring 
//===================================================

#pragma namespace ("\\\\.\\root\\cimv2")
#pragma deleteclass("Win32_BitLockerEncryptionDetails", NOFAIL) 
[Union, ViewSources{"select DeviceId, BitlockerPersistentVolumeId, BitLockerManagementPersistentVolumeId, BitLockerManagementVolumeType, DriveLetter, Compliant, ReasonsForNonCompliance, KeyProtectorTypes, EncryptionMethod, ConversionStatus, ProtectionStatus, IsAutoUnlockEnabled, NoncomplianceDetectedDate, EnforcePolicyDate from Mbam_Volume"}, ViewSpaces{"\\\\.\\root\\microsoft\\mbam"}, dynamic, Provider("MS_VIEW_INSTANCE_PROVIDER")]
class Win32_BitLockerEncryptionDetails
{
    [PropertySources{"DeviceId"},key]
    String     DeviceId;
    [PropertySources{"BitlockerPersistentVolumeId"}]
    String     BitlockerPersistentVolumeId;
    [PropertySources{"BitLockerManagementPersistentVolumeId"}]
    String     MbamPersistentVolumeId;
    //UNKNOWN = 0, OS_Volume = 1, FIXED_VOLUME = 2, REMOVABLE_VOLUME = 3
    [PropertySources{"BitLockerManagementVolumeType"}]
    SInt32     MbamVolumeType;
    [PropertySources{"DriveLetter"}]
    String     DriveLetter;
    //VOLUME_NOT_COMPLIANT = 0, VOLUME_COMPLIANT = 1, NOT_APPLICABLE = 2
    [PropertySources{"Compliant"}]
    SInt32     Compliant;
    [PropertySources{"ReasonsForNonCompliance"}]
    SInt32     ReasonsForNonCompliance[];
    [PropertySources{"KeyProtectorTypes"}]
    SInt32     KeyProtectorTypes[];
    [PropertySources{"EncryptionMethod"}]
    SInt32     EncryptionMethod;
    [PropertySources{"ConversionStatus"}]
    SInt32     ConversionStatus;
    [PropertySources{"ProtectionStatus"}]
    SInt32     ProtectionStatus;
    [PropertySources{"IsAutoUnlockEnabled"}]
    Boolean     IsAutoUnlockEnabled;
    [PropertySources{"NoncomplianceDetectedDate"}]
    String     NoncomplianceDetectedDate;
    [PropertySources{"EnforcePolicyDate"}]
    String     EnforcePolicyDate;
};

#pragma namespace ("\\\\.\\root\\cimv2")
#pragma deleteclass("Win32Reg_MBAMPolicy", NOFAIL)
[DYNPROPS]
Class Win32Reg_MBAMPolicy
{
    [key]
    string KeyName;
    
    //General encryption requirements
    UInt32    OsDriveEncryption;
    UInt32    FixedDataDriveEncryption;
    UInt32    EncryptionMethod;
    
    //Required protectors properties
    UInt32    OsDriveProtector;
    UInt32    FixedDataDriveAutoUnlock;
    UInt32    FixedDataDrivePassphrase;

    //MBAM Agent fields
    Uint32    MBAMPolicyEnforced;
    string    LastConsoleUser;
    datetime  UserExemptionDate;
    UInt32    MBAMMachineError;

    // Encoded Computer Name
    string    EncodedComputerName;
};

[DYNPROPS]
Instance of Win32Reg_MBAMPolicy
{
    KeyName="BitLocker policy";
    
    //General encryption requirements
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Microsoft\\FVE\\MDOPBitLockerManagement|ShouldEncryptOsDrive"),Dynamic,Provider("RegPropProv")]
    OsDriveEncryption;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Microsoft\\FVE\\MDOPBitLockerManagement|ShouldEncryptFixedDataDrive"),Dynamic,Provider("RegPropProv")]
    FixedDataDriveEncryption;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Microsoft\\FVE|EncryptionMethod"),Dynamic,Provider("RegPropProv")]
    EncryptionMethod;
    
    //Required protectors properties
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|OSVolumeProtectorPolicy"),Dynamic,Provider("RegPropProv")]
    OsDriveProtector;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Microsoft\\FVE\\MDOPBitLockerManagement|AutoUnlockFixedDataDrive"),Dynamic,Provider("RegPropProv")]
    FixedDataDriveAutoUnlock;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Microsoft\\FVE|FDVPassphrase"),Dynamic,Provider("RegPropProv")]
    FixedDataDrivePassphrase;

    //MBAM agent fields
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|MBAMPolicyEnforced"),Dynamic,Provider("RegPropProv")]
    MBAMPolicyEnforced;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|LastConsoleUser"),Dynamic,Provider("RegPropProv")]
    LastConsoleUser;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|UserExemptionDate"),Dynamic,Provider("RegPropProv")]
    UserExemptionDate; //Registry value should be string in the format of yyyymmddHHMMSS.mmmmmmsUUU
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|MBAMMachineError"),Dynamic,Provider("RegPropProv")]
    MBAMMachineError;
    [PropertyContext("Local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\MBAM|EncodedComputerName"),Dynamic,Provider("RegPropProv")]
    EncodedComputerName;
};

#pragma namespace ("\\\\.\\root\\cimv2")
#pragma deleteclass("CCM_OperatingSystemExtended", NOFAIL)
[Union, ViewSources{"select Name,OperatingSystemSKU from Win32_OperatingSystem"}, ViewSpaces{"\\\\.\\root\\cimv2"},
dynamic,Provider("MS_VIEW_INSTANCE_PROVIDER")]
class CCM_OperatingSystemExtended
{
    [PropertySources{"Name"},key]
    string     Name;
    [PropertySources{"OperatingSystemSKU"}]
    uint32     SKU;
};

#pragma namespace ("\\\\.\\root\\cimv2")
#pragma deleteclass("CCM_ComputerSystemExtended", NOFAIL)
[Union, ViewSources{"select Name,PCSystemType from Win32_ComputerSystem"}, ViewSpaces{"\\\\.\\root\\cimv2"},
dynamic,Provider("MS_VIEW_INSTANCE_PROVIDER")]
class CCM_ComputerSystemExtended
{
    [PropertySources{"Name"},key]
    string     Name;
    [PropertySources{"PCSystemType"}]
    uint16     PCSystemType;
};

//=======================================================
// Microsoft BitLocker Administration and Monitoring end
//=======================================================
```