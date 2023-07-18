# Initial Enumeration

Target is to escalate our privilege to one of the following depending on the system configuration and what type of data we encounter

* `NT AUTHORITY\SYSTEM` and LocalSystem account -> more privileged than local admin.
* Local administrator
* Account that is member of local `Administrators` group -> has same privilege as local admin
* Domain user part of local `Administrators` group
* Domain admin (high privilege account in AD) that is part of Administrators group

### Key Data Points

* **OS name** -> what type of Windows, this will give us info needed to select tools.
* **Version** -> give us info to find public exploits (may crash the system, understand before run)
* **Running Services** -> Espeacially those running as NT AUTHORITY\SYSTEM or admin

### System Information

**Tasklist**

Gives a better idea of what applications are currently running on the system.

```cmd-session
C:\htb> tasklist /svc
```

Get familiar with standard Windows processes

* [Session Manager Subsystem (smss.exe)](https://en.wikipedia.org/wiki/Session\_Manager\_Subsystem)
* [Client Server Runtime Subsystem (csrss.exe)](https://en.wikipedia.org/wiki/Client/Server\_Runtime\_Subsystem)
* [WinLogon (winlogon.exe)](https://en.wikipedia.org/wiki/Winlogon)
* [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)
* [Service Host (svchost.exe)](https://en.wikipedia.org/wiki/Svchost.exe)

**Display All Environment Variables**

```cmd-session
C:\htb> set

ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Administrator\AppData\Roaming
CommonProgramFiles=C:\Program Files\Common Files
. . .
```

Common attack example:

* place .py or .jar in path to execute
* if folder in PATH is writable, perform DLL Injection
  * Windows look for file in current directory then in PATH (reading left to right)

**View Detailed Configuration Information**

```cmd-session
C:\htb> systeminfo

Host Name:                 WINLPE-SRV01
OS Name:                   Microsoft Windows Server 2016 Standard
OS Version:                10.0.14393 N/A Build 14393
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
. . .
```

**Patches and Updates**

```cmd-session
C:\htb> wmic qfe

Caption                                     CSName        Description      FixComments  HotFixID   InstallDate  InstalledBy          InstalledOn  Name  ServicePackInEffect  Status
http://support.microsoft.com/?kbid=3199986  WINLPE-SRV01  Update                        KB3199986               NT AUTHORITY\SYSTEM  11/21/2016
https://support.microsoft.com/help/5001078  WINLPE-SRV01  Security Update               KB5001078               NT AUTHORITY\SYSTEM  3/25/2021
http://support.microsoft.com/?kbid=4103723  WINLPE-SRV01  Security Update               KB4103723               NT AUTHORITY\SYSTEM  3/25/2021
```

```powershell-session
PS C:\htb> Get-HotFix | ft -AutoSize

Source       Description     HotFixID  InstalledBy                InstalledOn
------       -----------     --------  -----------                -----------
WINLPE-SRV01 Update          KB3199986 NT AUTHORITY\SYSTEM        11/21/2016 12:00:00 AM
WINLPE-SRV01 Update          KB4054590 WINLPE-SRV01\Administrator 3/30/2021 12:00:00 AM
WINLPE-SRV01 Security Update KB5001078 NT AUTHORITY\SYSTEM        3/25/2021 12:00:00 AM
WINLPE-SRV01 Security Update KB3200970 WINLPE-SRV01\Administrator 4/13/2021 12:00:00 AM
```

**Installed Programs**

```cmd-session
C:\htb> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.24.28127
Java 8 Update 231 (64-bit)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.24.28127
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127
. . .
```

```powershell-session
PS C:\htb> Get-WmiObject -Class Win32_Product |  select Name, Version

Name                                                                    Version
----                                                                    -------
SQL Server 2016 Database Engine Shared                                  13.2.5026.0
Microsoft OLE DB Driver for SQL Server                                  18.3.0.0
Microsoft Visual C++ 2010  x64 Redistributable - 10.0.40219             10.0.40219
Microsoft Help Viewer 2.3                                               2.3.28107
```

Then run LaZagne and look for stored credentials on installed software (FileZilla, Putty, etc)

**Display Running Processes**

```cmd-session
PS C:\htb> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       1096
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       840
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:1433           0.0.0.0:0              LISTENING       3520
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       968
<...SNIP...>
```

### User & Group Information

**Logged-In Users**

```cmd-session
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#2           1  Active          .  3/25/2021 9:27 AM
```

**Current User**

```cmd-session
C:\htb> echo %USERNAME%
C:\htb> #OR
C:\htb> whoami
```

**Current User Privileges**

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

If we have service account we may have `SeImpersonatePrivilege` set and could be abused using **Juicy Potato** or **God Potato**

**Current User Group Information**

```cmd-session
C:\htb> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\REMOTE INTERACTIVE LOGON  Well-known group S-1-5-14     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
```

**Get All Users**

```cmd-session
C:\htb> net user

User accounts for \\WINLPE-SRV01

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
helpdesk                 htb-student              jordan
sarah                    secsvc
The command completed successfully.
```

We may find scripts with passwords or SSH keys so it's worth noting other users present on the system.

**Get All Groups**

```cmd-session
C:\htb> net localgroup

Aliases for \\WINLPE-SRV01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*Backup Operators
*Certificate Service DCOM Access
*Cryptographic Operators
. . .
```

**Details About a Group**

```cmd-session
C:\htb> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
helpdesk
sarah
secsvc
The command completed successfully.
```

We may find passwords of other low priv users in comments which could be leveraged to gain privilege escalation.

**Get Password Policy & Other Account Information**

```cmd-session
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          42
Minimum password length:                              0
Length of password history maintained:                None
Lockout threshold:                                    Never
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

## Assessment

**What non-default privilege does the htb-student user have?**

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Run normally</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>Run as admin</p></figcaption></figure>

* SeTakeOwnershipPrivilege

**Who is a member of the Backup Operators group?**

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption><p>sarah</p></figcaption></figure>

**What service is listening on port 8080 (service name not the executable)?**

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption><p>Tomcat8</p></figcaption></figure>

1. Get pid using `netstat -aof | findstr :8080`&#x20;
2. Get taskname using `tasklist /fi "pid eq 2320"` (2320 is the pid got from step 1)

**What user is logged in to the target host?**

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption><p>sccm_svc</p></figcaption></figure>

**What type of session does this user have?**

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>console</p></figcaption></figure>
