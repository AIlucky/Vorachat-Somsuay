# Kerberoasting

Kerberoasting is a privilege escalation technique in AD environments, which attakc targets Service Principal Names (SPN) accounts.

SPN is a unique id that Kerberos use to map service with service account (whose context the service is running.)

* **Any domain user can request a Kerberos ticket** for any service account in the same domain
* All you need to perform a Kerberoasting attack are one of the following
  * account's cleartext password (or NTLM hash)
  * a shell in the context of a domain user account
  * SYSTEM level access on a domain-joined host
* Domain accounts running services are often local administrators
* Retriving a Kerberos ticket for account with SPN does not allow to execute commands in the context of the account
* TGS-REP is encrypted with service account's NTLM hash
  * So the hash could be cracked locally (offline) using tools like hashcat
* Service accounts are often configured with weak or reused password to simplify administration
* Sometimes the password is the same as the username
* Cracking a ticket obtained via a Kerberoasting attack gives a low-privilege user account
* TGS tickets take longer to crack than other formats such as NTLM hashes
* Kerberoasting and the presence of SPNs do not guarantee us any level of access

### Kerberoasting - Performing the Attack

Depending on your position in a network, this attack can be performed in multiple ways:

* From a non-domain joined Linux host using valid domain user credentials.
* From a domain-joined Linux host as root after retrieving the keytab file.
* From a domain-joined Windows host authenticated as a domain user.
* From a domain-joined Windows host with a shell in the context of a domain account.
* As SYSTEM on a domain-joined Windows host.
* From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.

Tools:

* Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
* A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
* From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.
