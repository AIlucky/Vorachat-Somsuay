---
description: If DC is Virtualized then this group should be considered Domain Admins
---

# Hyper-V Administrators

{% embed url="https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/" %}
Check This blog for more info
{% endembed %}

In the blog it states that when deleting VM, vmms.exe trys to restore the original file permissions on .vhdx file and does that as NT AUTHORITY/SYSTEM. We can delete .vhdx file and create a hard link to point this file to a protected SYSTEM file, which we have full permissions to.

[CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) or [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841) Can be used to gain SYSTEM privilege. Or, we can run app that has a installed service running as SYSTEM and startable by low priv users.

**Target File**

```shell-session
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

Firefox is good example which installs `Mozilla Maintenance Service`

{% embed url="https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1" %}
Use this exploit to grant our current user full permissions on the file above
{% endembed %}

**Taking Ownership of the File**

```cmd-session
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

**Starting the Mozilla Maintenance Service**

Replace this file with a malicious `maintenanceservice.exe`

```cmd-session
C:\htb> sc.exe start MozillaMaintenance
```



