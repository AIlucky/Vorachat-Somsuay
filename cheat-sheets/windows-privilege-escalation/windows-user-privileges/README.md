# Windows User Privileges

* Privilege and Access rights are different things
* User and Group privilege are stored in secure database and access token is granted everytime the user authenticates
* If user performs any privileged action, the access token is reviewed if it has the right privilege.
* A user can have different Privilege in local system and other system across the domain

### Windows Authorization Process

`Security principals` -> anything that can be **authenticated** by the OS

* It is the primary way of limiting access on Windows hosts
* `Security principals` are identified using unique SID
* Once `Security principal` is created, it gets a unique SID and remains to that principal for it's lifetime

![Diagram of the Windows authorization and access control process.](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/media/authorization-and-access-control-process.png)

### Rights and Privileges in Windows

Some of the groups are listed as following

<table data-header-hidden><thead><tr><th width="161"></th><th></th></tr></thead><tbody><tr><td><strong>Group</strong></td><td><strong>Description</strong></td></tr><tr><td>Default Administrators</td><td>Domain Admins and Enterprise Admins are <strong>"super" groups.</strong></td></tr><tr><td>Server Operators</td><td>Members can <strong>modify services, access SMB shares, and backup files</strong>.</td></tr><tr><td>Backup Operators</td><td>Members are allowed to log onto DCs locally and should be <strong>considered Domain Admins</strong>. They can <strong>make shadow copies of the SAM/NTDS</strong> database, <strong>read the registry</strong> <strong>remotely</strong>, and <strong>access</strong> <strong>file system on</strong> <strong>DC via SMB</strong>. This group is sometimes added to the local Backup Operators group on non-DCs.</td></tr><tr><td>Print Operators</td><td>Members can log on to DCs locally and <strong>"trick" Windows into loading a malicious driver</strong>.</td></tr><tr><td>Hyper-V Administrators</td><td>If there are <em><strong>virtual DCs</strong></em>, any virtualization admins, such as <strong>members of Hyper-V Administrators, should be considered Domain Admins</strong>.</td></tr><tr><td>Account Operators</td><td>Members can <strong>modify non-protected accounts and groups</strong> in the domain.</td></tr><tr><td>Remote Desktop Users</td><td>Members are not given any useful permissions by default but <strong>are often granted additional rights</strong> such as <code>Allow Login Through Remote Desktop Services</code> and <strong>can move laterally using the RDP protocol</strong>.</td></tr><tr><td>Remote Management Users</td><td>Members can <strong>log on to DCs with PSRemoting</strong> (This group is sometimes added to the local remote management group on non-DCs).</td></tr><tr><td>Group Policy Creator Owners</td><td>Members can create new GPOs but would need to be delegated additional permissions to link GPOs to a container such as a domain or OU.</td></tr><tr><td>Schema Admins</td><td>Members can <strong>modify the Active Directory schema structure</strong> and backdoor any to-be-created Group/GPO by <strong>adding a compromised account to the default object ACL</strong>.</td></tr><tr><td>DNS Admins</td><td>Members can <strong>load a DLL on a DC</strong>, but do not have the necessary permissions to restart the DNS server. They can load a malicious DLL and wait for a reboot as a persistence mechanism. Loading a DLL will often result in the service crashing. A more reliable way to exploit this group is to <a href="https://cube0x0.github.io/Pocing-Beyond-DA/">create a WPAD record</a>.</td></tr></tbody></table>

### User Rights Assignment

{% embed url="https://4sysops.com/archives/user-rights-assignment-in-windows-server-2016/" %}
key user rights assignments
{% endembed %}

* `whoami /priv` -> will give a list of all user rights
* **Some rights are only listed when running an elevated cmd or PowerShell session.**

**Local Admin User Rights - Elevated**

```powershell-session
PS C:\htb> whoami 

winlpe-srv01\administrator


PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
```

* `Disabled` means our account has specific privilege assigned and cannot be used in access token until enabled.
* PowerShell and cmd can't enable these, we have to run a script that does so.

{% embed url="https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts%5CEnable-Privilege.ps1" %}

{% embed url="https://www.leeholmes.com/adjusting-token-privileges-in-powershell/" %}

**Standard User Rights**

```powershell-session
PS C:\htb> whoami 

winlpe-srv01\htb-student


PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

**Backup Operators Rights**

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```
