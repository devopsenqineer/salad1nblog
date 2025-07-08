---
layout: post
title: sysprep for automation
description: automation using sysprep
summary: sysprep
tags: automation
minute: 7
lang: en
---

# How to use sysprep for automation?

### What is sysprep actually?

Sysprep stands for system preparation. It is a tool that is integrated within Windows operating systems. It prepares a Windows client or a Windows server to remove specific information from the Windows operating system, e.g. the existing SID is deleted. By running sysprep, the image can be used to be installed on multiple instances. 

### Which functions are good about sysprep?

The following functions are very helpful:

- **Removes specific information**: It removes specific information and especially the SID. 

- **Uninstalls drivers**: Uninstalls drivers, but does not remove them

- **Add an answer file (unattend)**: Allows you to add an answer file to the existing operating system.

### And how can I use sysprep for automation now?

First, I'll give some context as to why you can and should use sysprep.
**Following scenario:**
Based on my Terraform project (https://github.com/devopsenqineer/terraform) which creates virtual machines in a Hyper-V environment, assigns the hostname and IP addresses through scripts, there was the following problem: The domain join was not possible because the machines used **all** had the same SID. As a result, I received an error message that this was not possible due to the multiple SID. 

***Possible solution***:

Through research, I came across the integrated tool sysprep. Through a conversation with a colleague at work, the possibility of sysprep was explained to me in more detail, as I was not familiar with it at the time. 

I continued to read and familiarize myself with sysprep and needed several test runs to get the desired result. In addition, it was previously difficult to activate WinRM (Windows Remote Management). 

I had to use a Powershell script to activate WinRM which can be found under the link (https://gist.github.com/tdigangi5/85b0b3bb5f7b35c7b829828928f22cdd). Since I had to run this script every time on a new machine to activate WinRM, it was worth considering putting it in an answer file. 

Through intensive research I found parts for the answer file and will share them with you. This answer file can still be revised and further perfected. 

## 
**Explanation for the answer file:**

I will briefly explain how to use the answer file and sysprep:

- **1st step**: Change the user and password within the answer file, and adjust the path for the PowerShell script for WinRM. 

- **2nd step:** Then save the answer file in the sysprep folder. I have named the answer file deploy.xml. 

    The path is as follows: **"C:\Windows\System32\Sysprep"**.

- Step 3:** The last step is to open a cmd window and enter the following command:

    
    ```cmd
    C:\Windows\System32\Sysprep\sysprep.exe /generalize /shutdown /oobe /unattend: "C:\Windows\System32\Sysprep\deploy.xml" 
    ```

- **4th step:** As the last step and actually one of the **most important** steps: Do **NOT** boot the virtual machine. The virtual machine should now be exported and only started up after successful export and, if necessary, copying of the virtual machine. 

## Code of the answer file

```xml
<?xml version="1.0" encoding="UTF-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
   <settings pass="oobeSystem">
      <component xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
         <UserAccounts>
            <AdministratorPassword>
               <Value>Password12345</Value>
               <PlainText>true</PlainText>
            </AdministratorPassword>
         </UserAccounts>
         <AutoLogon>
            <Username>Administrator</Username>
            <Enabled>true</Enabled>
            <LogonCount>1</LogonCount>
            <Password>
               <Value>Password12345</Value>
               <PlainText>true</PlainText>
            </Password>
         </AutoLogon>
         <FirstLogonCommands>
            <SynchronousCommand wcm:action="add">
               <Order>1</Order>
               <CommandLine>reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoLogonCount /t REG_DWORD /d 0 /f</CommandLine>
            </SynchronousCommand>
            <SynchronousCommand wcm:action="add">
               <Order>2</Order>
               <CommandLine>powershell.exe -File "C:\{PathToYourScriptOnTheHost}\winrm.ps1"</CommandLine>
            </SynchronousCommand>
         </FirstLogonCommands>
         <OOBE>
            <HideEULAPage>true</HideEULAPage>
            <HideLocalAccountScreen>true</HideLocalAccountScreen>
            <HideOEMRegistrationScreen>true</HideOEMRegistrationScreen>
            <HideOnlineAccountScreens>true</HideOnlineAccountScreens>
            <HideWirelessSetupInOOBE>true</HideWirelessSetupInOOBE>
            <NetworkLocation>Other</NetworkLocation>
            <ProtectYourPC>3</ProtectYourPC>
            <SkipMachineOOBE>true</SkipMachineOOBE>
            <SkipUserOOBE>true</SkipUserOOBE>
            <UnattendEnableRetailDemo>false</UnattendEnableRetailDemo>
         </OOBE>
      </component>
   </settings>
</unattend>
```

Thanks for reading my blog.
