---
description: >-
  SharpHound/BloodHound, PowerView/SharpView, Grouper2, Snaffler, and some
  built-in tools
---

# Credentialed Enumeration - from Windows

## ActiveDirectory PowerShell Module

a group of PowerShell cmdlets for administering an Active Directory environment.

```
PS C:\htb> Get-Module # list modules that are imported
PS C:\htb> Import-Module ActiveDirectory
```

### Get Domain Info

```powershell-session
PS C:\htb> Get-ADDomain
```

This gives us info such as domain SID, domain functional level, child domains, etc. If we want to filter for accounts that has ServicePrincipalName property, use the following command.

### **Get-ADUser**

```powershell-session
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

### Get-ADTrust

Checking For Trust Relationships, this will give us any trust relationshops the domain has. (useful for attacking across forest)

```powershell-session
PS C:\htb> Get-ADTrust -Filter *
```

We can take the results and feed interesting names back into the cmdlet to get more detailed information about a particular group like so:

```powershell-session
PS C:\htb> Get-ADGroup -Identity "Backup Operators"

DistinguishedName : CN=Backup Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
GroupCategory     : Security
GroupScope        : DomainLocal
Name              : Backup Operators
ObjectClass       : group
ObjectGUID        : 6276d85d-9c39-4b7c-8449-cad37e8abc38
SamAccountName    : Backup Operators
SID               : S-1-5-32-551
```

## PowerView

like **BloodHound**, it provides a way to identify where users are logged in on a network, enumerate domain information such as users, computers, groups, ACLS, trusts, hunt for file shares and passwords, perform Kerberoasting, and more.

### List of common PowerView commands

<table data-header-hidden><thead><tr><th width="330"></th><th></th></tr></thead><tbody><tr><td><strong>Command</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Export-PowerViewCSV</code></td><td>Append results to a CSV file</td></tr><tr><td><code>ConvertTo-SID</code></td><td>Convert a User or group name to its SID value</td></tr><tr><td><code>Get-DomainSPNTicket</code></td><td>Requests the Kerberos ticket for a specified Service Principal Name (SPN) account</td></tr><tr><td><strong>Domain/LDAP Functions:</strong></td><td></td></tr><tr><td><code>Get-Domain</code></td><td>Will return the AD object for the current (or specified) domain</td></tr><tr><td><code>Get-DomainController</code></td><td>Return a list of the Domain Controllers for the specified domain</td></tr><tr><td><code>Get-DomainUser</code></td><td>Will return all users or specific user objects in AD</td></tr><tr><td><code>Get-DomainComputer</code></td><td>Will return all computers or specific computer objects in AD</td></tr><tr><td><code>Get-DomainGroup</code></td><td>Will return all groups or specific group objects in AD</td></tr><tr><td><code>Get-DomainOU</code></td><td>Search for all or specific OU objects in AD</td></tr><tr><td><code>Find-InterestingDomainAcl</code></td><td>Finds object ACLs in the domain with modification rights set to non-built in objects</td></tr><tr><td><code>Get-DomainGroupMember</code></td><td>Will return the members of a specific domain group</td></tr><tr><td><code>Get-DomainFileServer</code></td><td>Returns a list of servers likely functioning as file servers</td></tr><tr><td><code>Get-DomainDFSShare</code></td><td>Returns a list of all distributed file systems for the current (or specified) domain</td></tr><tr><td><strong>GPO Functions:</strong></td><td></td></tr><tr><td><code>Get-DomainGPO</code></td><td>Will return all GPOs or specific GPO objects in AD</td></tr><tr><td><code>Get-DomainPolicy</code></td><td>Returns the default domain policy or the domain controller policy for the current domain</td></tr><tr><td><strong>Computer Enumeration Functions:</strong></td><td></td></tr><tr><td><code>Get-NetLocalGroup</code></td><td>Enumerates local groups on the local or a remote machine</td></tr><tr><td><code>Get-NetLocalGroupMember</code></td><td>Enumerates members of a specific local group</td></tr><tr><td><code>Get-NetShare</code></td><td>Returns open shares on the local (or a remote) machine</td></tr><tr><td><code>Get-NetSession</code></td><td>Will return session information for the local (or a remote) machine</td></tr><tr><td><code>Test-AdminAccess</code></td><td>Tests if the current user has administrative access to the local (or a remote) machine</td></tr><tr><td><strong>Threaded 'Meta'-Functions:</strong></td><td></td></tr><tr><td><code>Find-DomainUserLocation</code></td><td>Finds machines where specific users are logged in</td></tr><tr><td><code>Find-DomainShare</code></td><td>Finds reachable shares on domain machines</td></tr><tr><td><code>Find-InterestingDomainShareFile</code></td><td>Searches for files matching specific criteria on readable shares in the domain</td></tr><tr><td><code>Find-LocalAdminAccess</code></td><td>Find machines on the local domain where the current user has local administrator access</td></tr><tr><td><strong>Domain Trust Functions:</strong></td><td></td></tr><tr><td><code>Get-DomainTrust</code></td><td>Returns domain trusts for the current domain or a specified domain</td></tr><tr><td><code>Get-ForestTrust</code></td><td>Returns all forest trusts for the current forest or a specified forest</td></tr><tr><td><code>Get-DomainForeignUser</code></td><td>Enumerates users who are in groups outside of the user's domain</td></tr><tr><td><code>Get-DomainForeignGroupMember</code></td><td>Enumerates groups with users outside of the group's domain and returns each foreign member</td></tr><tr><td><code>Get-DomainTrustMapping</code></td><td>Will enumerate all trusts for the current domain and any others seen.</td></tr></tbody></table>

### Get-DomainUser

This will provide us with information on all users or specific users we specify

```powershell-session
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

### Get-DomainGroupMember

to retrieve group-specific information, using this command we will know who to target for potential elevation of privileges.

```powershell-session
PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

* \-Recurse -> if find any group that are part of target group, list the members of those groups.

### Get-DomainTrustMapping

```powershell-session
PS C:\htb> Get-DomainTrustMapping
```

### Test-AdminAccess

to test for local admin access on either the current machine or a remote one.

```powershell-session
PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01

ComputerName    IsAdmin
------------    -------
ACADEMY-EA-MS01    True 
```

we are currently using is an administrator on the host ACADEMY-EA-MS01.&#x20;

Now we can check for users with the **SPN** attribute set, which indicates that **the account may be subjected to a Kerberoasting attack**.

### **Finding Users With SPN Set**

```powershell-session
PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName

serviceprincipalname                          samaccountname
--------------------                          --------------
adfsconnect/azure01.inlanefreight.local       adfs
backupjob/veam001.inlanefreight.local         backupagent
d0wngrade/kerberoast.inlanefreight.local      d0wngrade
kadmin/changepw                               krbtgt
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 sqldev
MSSQLSvc/SPSJDB.inlanefreight.local:1433      sqlprod
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351 sqlqa
sts/inlanefreight.local                       solarwindsmonitor
testspn/kerberoast.inlanefreight.local        testspn
testspn2/kerberoast.inlanefreight.local       testspn2
```

## SharpView

A .NET port of PowerView. Many of the same functions supported by PowerView can be used with SharpView. We can type a method name with `-Help` to get an argument list.

{% code overflow="wrap" %}
```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Help

Get_DomainUser -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -SPN <Boolean> -AdminCount <Boolean> -AllowDelegation <Boolean> -DisallowDelegation <Boolean> -TrustedToAuth <Boolean> -PreauthNotRequired <Boolean> -KerberosPreauthNotRequired <Boolean> -NoPreauth <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum> 
```
{% endcode %}

### Get-DomainUser

to enumerate information about a specific user

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend
```

## Shares

Allow user on doman to access info relevant to their roles and share content with organization

* Domain shares require user to be domain joined and authentication be fore access the system
* Permissions are also set to ensure user can access only allowed info
* Overly permissive shares can cause accidental disclosure of sensitive information
* We want to identify issues like thease to ensure no data is exposed to no priv users
* We can use PowerView to find shares and hunt for common strings. eg. `pass` in the name
* Snaffler can help use with the tedious process.

## Snaffler

At tool that help us acquire credentials or other sensitive data in AD. Snaffler obtains a list of hosts in the domain and enumerate those hosts for shares and readable directories. Then it iterates through any directory readable by our user and hunts for files that could serve to better our position within the assessment.

**Snaffler requires** that it be run from a **domain-joined host** or in a **domain-user context**.

```powershell-session
PS C:\htb> .\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data
```

* \-s -> print results to console
* \-d -> domain to search
* \-o -> write results to logfile (-o output.log)
* \-v data -> verbosity level (only display results on screen)

<figure><img src="../../../.gitbook/assets/image (79) (2).png" alt=""><figcaption><p>snaffler in action</p></figcaption></figure>

We may find passwords, SSH keys, configuration files, or other data that can be used to further our access.

## BloodHound

Authenticate as domain user from windows attack host in the network (but not joined to the domain) or transfer the tool to a domain-joined host

running the SharpHound.exe collector from the MS01 attack host.

```powershell-session
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT
```

<figure><img src="../../../.gitbook/assets/image (80) (1).png" alt=""><figcaption></figcaption></figure>

**To run blood hound on our attack machine**

1. Transfer the output file from sharp hound to our attack machine.
2. Type bloodhound in the terminal, CMD, or PowerShell console.
3. Type in the credentials (default is neo4j:neo4j)
4. Click upload data on the right
5. Upload the zip file and click open
6. Close the upload window.

On the search bar we can type domain: and select the domain we want from the results. Will see the information displayed to us.

`Find Computers with Unsupported Operating Systems`  function in the analysis tabe is great for finding outdated and unsupported OS running old software. These hose may be susceptible to RCE vulnerabilities like [MS08-067](https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15).

![Unsupported Operating Systems](https://academy.hackthebox.com/storage/modules/143/unsupported.png)

This query shows two hosts, one running Windows 7 and one running Windows Server 2008 (both of which are not "live" in our lab). Sometimes we will see hosts that are no longer powered on but still appear as records in AD.

`Find Computers where Domain Users are Local Admin` to quickly see if there are any hosts where all users have local admin rights. If this is the case, then any account we control can typically be used to access the host(s) in question, and we may be able to retrieve credentials from memory or find other sensitive data.

![Local Admins](https://academy.hackthebox.com/storage/modules/143/local-admin.png)

## Assessment

Using Bloodhound, determine how many Kerberoastable accounts exist within the INLANEFREIGHT domain.

Collect information to be passed into bloodhound

```
PS C:\htb> .\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p>13 Kerberoastable Accounts</p></figcaption></figure>

**What PowerView function allows us to test if a user has administrative access to a local or remote host?**

* Test-AdminAccess

**Run Snaffler and hunt for a readable web config file. What is the name of the user in the connection string within the file?**

```
.\Snaffler.exe -d INLANEFREIGHT.LOCAL -s -v data
 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler


2023-07-03 21:40:19 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\ADMIN$)
2023-07-03 21:40:19 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\C$)
2023-07-03 21:40:19 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares)
2023-07-03 21:40:19 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\User Shares)
2023-07-03 21:40:19 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\ZZZ_archive)
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.kdb$|289B|3/31/2022 12:09:22 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\GroupBackup.kdb) .kdb
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.keychain$|295B|3/31/2022 12:08:42 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\SetStep.keychain) .keychain
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.ppk$|275B|3/31/2022 12:04:40 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\StopTrace.ppk) .ppk
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.sqldump$|310B|3/31/2022 12:05:02 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\AddPublish.sqldump) .sqldump
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.sqldump$|312B|3/31/2022 12:05:30 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\DenyRedo.sqldump) .sqldump
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.keypair$|278B|3/31/2022 12:09:09 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\UnprotectConvertTo.keypair) .keypair
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.key$|301B|3/31/2022 12:09:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WaitClear.key) .key
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.key$|299B|3/31/2022 12:05:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ShowReset.key) .key
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.tblk$|279B|3/31/2022 12:05:25 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FindConnect.tblk) .tblk
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.tblk$|280B|3/31/2022 12:05:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\ExportJoin.tblk) .tblk
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.key$|298B|3/31/2022 12:05:10 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ProtectStep.key) .key
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.kwallet$|302B|3/31/2022 12:04:45 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WriteUse.kwallet) .kwallet
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.mdf$|305B|3/31/2022 12:09:27 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FormatShow.mdf) .mdf
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.tblk$|301B|3/31/2022 12:05:15 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\SubmitConvertFrom.tblk) .tblk
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepConfigRegexRed|RW|connectionstring[[:space:]]*=[[:space:]]*[\'\"][^\'\"].....*|253B|3/31/2022 12:12:43 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\web.config) <?xml version="1.0" encoding="utf-8"?>
<configuration>
  <connectionStrings>
    <add name="myConnectionString" connectionString="server=ACADEMY-EA-DB01;database=Employees;uid=sa;password=ILFREIGHTDB01!;" />
  </connectionStrings>
</configuration>
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.psafe3$|301B|3/31/2022 12:09:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\GetUpdate.psafe3) .psafe3
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.mdf$|299B|3/31/2022 12:09:14 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\LockConfirm.mdf) .mdf
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.sqldump$|302B|3/31/2022 12:08:58 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\RestoreCopy.sqldump) .sqldump
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.ova$|306B|3/31/2022 12:08:45 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\NewCopy.ova) .ova
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.sqldump$|318B|3/31/2022 12:09:01 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\ResolveExpand.sqldump) .sqldump
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.cscfg$|309B|3/31/2022 12:04:57 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\RequestUnprotect.cscfg) .cscfg
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.psafe3$|275B|3/31/2022 12:08:50 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\RedoPop.psafe3) .psafe3
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.psafe3$|305B|3/31/2022 12:09:41 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\WriteStart.psafe3) .psafe3
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.vhdx$|282B|3/31/2022 12:05:07 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\DB01-BACKUP.vhdx) .vhdx
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.wim$|289B|3/31/2022 12:09:38 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\NewOptimize.wim) .wim
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.vhdx$|285B|3/31/2022 12:09:11 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\WEB01-OLD.vhdx) .vhdx
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepExtExactRed|RW|^\.pcap$|279B|3/31/2022 12:04:55 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\UnprotectRestart.pcap) .pcap
2023-07-03 21:40:47 -07:00 [File] {Black}<KeepExtExactBlack|RW|^\.ovpn$|288B|3/31/2022 12:05:20 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Systems\StepSync.ovpn) .ovpn
Press any key to exit.
```

focus on the following

```
2023-07-03 21:40:47 -07:00 [File] {Red}<KeepConfigRegexRed|RW|connectionstring[[:space:]]*=[[:space:]]*[\'\"][^\'\"].....*|253B|3/31/2022 12:12:43 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\web.config) <?xml version="1.0" encoding="utf-8"?>
<configuration>
  <connectionStrings>
    <add name="myConnectionString" connectionString="server=ACADEMY-EA-DB01;database=Employees;uid=sa;password=ILFREIGHTDB01!;" />
  </connectionStrings>
</configuration>
```

The user insied the connection string is `sa:ILFREIGHTDB01!`

**What is the password for the database user?**

* `sa:ILFREIGHTDB01!`

