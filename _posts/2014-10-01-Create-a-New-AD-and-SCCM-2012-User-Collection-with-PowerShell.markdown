---
published: true
title: Create a New AD and SCCM 2012 User Collection with PowerShell
layout: post
---
Function New-SCCMUserCollection {
 PARAM (
  [Parameter(Mandatory = $true)] $ADCollectionName,
                [Parameter(Mandatory = $true)] $CMCollectionName,
  $limitingCollection  = “All Users”,
## Customize the $path appropriately for your environment.
  $path    = “OU=Application Deployment,OU=Groups,DC=,DC=”,
  $description   = “Application Deployment Group”,
## Customize your domain name in the query.
  $queryExpression  = “select SMS_R_USER.ResourceID,SMS_R_USER.ResourceType,SMS_R_USER.Name,SMS_R_USER.UniqueUserName,SMS_R_USER.WindowsNTDomain from SMS_R_User where SMS_R_User.SecurityGroupName = ‘\\$($ADCollectionName)'”,
  $ServerName  = “”,
  $SiteCode  = “<YourSiteCode"
 ) 
 ## Get a reference object for grabbing the RefreshSchedule property
        # Use an existing collection as a template to replicate the RefreshSchedule.
 $refreshSchedule = (Get-CMDeviceCollection -Name “7zip”).RefreshSchedule[0]
 $ruleName = $CMCollectionName
 
 ## Create AD Security Group
 New-ADGroup -Name $ADCollectionName -SamAccountName $ADCollectionName -GroupCategory Security -GroupScope Universal -DisplayName $ADCollectionName -Path $path -Description $description -Verbose
 
 ## Create SCCM Collection
 New-CMUserCollection -Name $CMCollectionName -LimitingCollectionName $limitingCollection -Verbose -RefreshSchedule $refreshSchedule -RefreshType 2
 
 ## Add collection rule
    $CollectionName = Get-CMUserCollection -Name $CMCollectionName
    Add-CMUserCollectionQueryMembershipRule -CollectionId $CollectionName.CollectionID -RuleName $ruleName -QueryExpression $queryExpression -Verbose
}

## Import the Configuration Manager 2012 Module
Import-Module “C:\Program Files (x86)\ConfigMgr2012\bin\ConfigurationManager.psd1”
## Set location the primary site code
Set-Location :
Import-Module ActiveDirectory

## Edit this variable to reflect the desired AD Security Group Names
$ADCollectionName = “Test User Collection”

## Do not edit this variable unless it is a special circumstance.
$CMCollectionName = “$ADCollectionName – User”

## Usage: New-SCCMUserCollection -ADCollectionName $ADCollectionName -CMCollectionName $CMCollectionName