# Windows Hardening guide
In this guide we will focus on the hardening of Windows to ensure security and (better) privacy for the end-user.

This guide will be split in 2 parts [Security](#security) and [Privacy](#privacy)

## Security
We will mainly focus on registry key because those can be implementened in the Home and Pro version.

- We expect in this guide that the main users is an standard user and that there is an seperate administrator account with admin privileges.
- In this guide we expect that your system has up-to-date drivers
- In this guide we expect that your system uses an **TPM3.0**
- Use only Microsoft/Windows Defender Anti-Virus and have no other 3th-party AV
- Your system should have an legit copy of Windows 11 and follow the system recommendations provided by Microsoft

#### We reccomend to implement the following if you meet the requirements above

1. [Attacksurface reduction](#attack-surface-reduction-rules) These are settings for Windows Defender to enhance the protection of your system.
2. [Virtualization Based Security](#virtualization-based-security) Isolates sensitive data using secure, isolated virtual environments.
3. [Anti-Malware Boot-Start Driver Policy](#anti-malware-boot-start-driver-policy) Manages boot drivers to ensure malware protection at startup.
4. Set [Bitlocker Drive Encryption](#bitlocker-drive-encryption) policies like forcing chipher strength and other settings related to Bitlocker.
5. Enable Bitlocker on the main system disk **(C:\\)** via Controle Panel.

#### Attack Surface Reduction rules
Group policy object: **Computer Configuration/System/Administrative Templates/Windows Components/Microsoft Defender Exploit Guard/Attack Surface Reduction/Configure Attack Surface Reduction rules**

If you run this [ASR.reg](/Windows/ASR.reg) it will implement the registry keys to set all the relevant [ASR rules](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference). This script will set all the ASR rules to "WARN"(6) this will give you the option to unblock the execution or process any time it runs.

Registry script:
```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender\Windows Defender Exploit Guard\ASR\Rules]
"56a863a9-875e-4185-98a7-b882c64b5ce5"="6" ; Block abuse of exploited vulnerable signed drivers.
"7674ba52-37eb-4a4f-a9a1-f0f9a1619a2c"="6" ; Block Adobe Reader from creating child processes.
"d4f940ab-401b-4efc-aadc-ad5f3c50688a"="6" ; Block all Office applications from creating child processes.
"9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2"="6" ; Block credential stealing from the Windows local security authority subsystem.
"be9ba2d9-53ea-4cdc-84e5-9b1eeee46550"="6" ; Block executable content from email client and webmail.
"01443614-cd74-433a-b99e-2ecdc07bfc25"="6" ; Block executable files from running unless they meet a prevalence, age, or trusted list criterion.
"5beb7efe-fd9a-4556-801d-275e5ffc04cc"="6" ; Block execution of potentially obfuscated scripts.
"d3e037e1-3eb8-44c8-a917-57927947596d"="6" ; Block JavaScript or VBScript from launching downloaded executable content.
"3b576869-a4ec-4529-8536-b80a7769e899"="6" ; Block Office applications from creating executable content.
"75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84"="6" ; Block Office applications from injecting code into other processes.
"26190899-1602-49e8-8b27-eb1d0a1ce869"="6" ; Block Office communication application from creating child processes.
"e6db77e5-3df2-4cf1-b95a-636979351e5b"="6" ; Block persistence through WMI event subscription.
"b2b3f03d-6a65-4f7b-a9c7-1c7ef74a9ba4"="6" ; Block untrusted and unsigned processes that run from USB.
"c0033c00-d16d-4114-a5a0-dc9b3a7d2ceb"="6" ; Block use of copied or impersonated system tools.
"92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b"="6" ; Block Win32 API calls from Office macros.
"c1db55ab-c21a-4637-bb3f-a12568109d35"="6" ; Use advanced protection against ransomware.
```
##### Making Exclusions
Some applicatios get blocked by ASR rules like driver update programs.
We reccomend using the Group policy editor to add exclusions.

Group policy object: **Computer Configuration/System/Administrative Templates/Windows Components/Microsoft Defender Exploit Guard/Attack Surface Reduction/Exclude files and paths from Attack Surface Reduction Rules**

### Virtualization Based Security
⚠️ ENSURE THAT **MEMORY INTEGRITY** IS SUCCESSFULY ENABLED BEFORE IMPLEMENTING VBS OTHERWISE THE MAHCINE WILL BOOTLOOP ⚠️

Group policy: **Computer Configuration/Administrative Templates/System/Device Guard/Turn On Virtualization Based Security**

#### This script below sets EUFI Lock, DMA protection & Audit mode
Registry script:
```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard]
"EnableVirtualizationBasedSecurity"=dword:00000001
"RequirePlatformSecurityFeatures"=dword:00000003
"HypervisorEnforcedCodeIntegrity"=dword:00000001
"HVCIMATRequired"=dword:00000001
"LsaCfgFlags"=dword:00000001
"MachineIdentityIsolation"=dword:00000001
"ConfigureSystemGuardLaunch"=dword:00000001
"ConfigureKernelShadowStacksLaunch"=dword:00000002
```

If any thing works and  there are no issues **(Check Event Viewer: Applications and Services Logs/Microsoft/Windows/DeviceGuard)** apply the script bellow to turn audit mode in to enforcement mode.

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard]
"EnableVirtualizationBasedSecurity"=dword:00000001
"RequirePlatformSecurityFeatures"=dword:00000003
"HypervisorEnforcedCodeIntegrity"=dword:00000001
"HVCIMATRequired"=dword:00000001
"LsaCfgFlags"=dword:00000001
"MachineIdentityIsolation"=dword:00000002
"ConfigureSystemGuardLaunch"=dword:00000001
"ConfigureKernelShadowStacksLaunch"=dword:00000001
```

### Anti-Malware Boot-Start Driver Policy
Group policy: **Computer Configuration/Administrative Templates/System/Early Launch Antimalware/Boot-Start Driver Initialization Policy**

This script allows "Good Only" so if there is any issues with 3th-party software or drivers check **Event Viewer: Applications and Services Logs/Microsoft/Windows/CodeIntegrity**

Registry script:
```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Policies\EarlyLaunch]
"DriverLoadPolicy"=dword:00000008
```

### Bitlocker Drive Encryption
Drive encryption is importand to setup strongly to ensure if your device is stolen or VM-disks are exfiltrated it is hard to gain access to the readable data.

Registry script:
```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\FVE]
"EncryptionMethodWithXtsRdv"=dword00000007 ; Removable Data Volumes set to XTS-AES 256
"EncryptionMethodWithXtsOs"=dword00000007 ; Operating System Volumes set to XTS-AES 256
"EncryptionMethodWithXtsFdv"=dword00000007 ; Fixed Data Volumes set to XTS-AES 256
```



## Privacy