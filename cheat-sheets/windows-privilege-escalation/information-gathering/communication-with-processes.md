---
description: Look for running processes in the system to gain access to those running them
---

# Communication with Processes

### Access Tokens

* Everytime user logs in to the system they authenticate and their password is verified with security database, if correct, they will get a token
* The copy of this token is used to determine the privilege level

### Enumerating Network Services

Similar to the [initial-enumeration.md](initial-enumeration.md "mention") section netstat command would give us the information of process ID and the port it's occupying

**Display Active Network Connections**

```cmd-session
C:\htb> netstat -ano
```

Look for:

* Entries listening on loopback address (127.0.0.1 and ::1)
  * Not listening on IP Address (10.129.43.8)
  * Not listening on broadcast (\*0.0.0.0, ::/0)

Example:

```cmd-session
TCP    127.0.0.1:14147        0.0.0.0:0              LISTENING       3812
```

\*\*network sockets on localhost are often insecure\*\*

Another Example:

1. Splunk did not have any authentication which allowed anyone to deploy applications leading to code execution (default config of Splunk was to run as SYSTEM$ not low priv user)
2. Erlang (25672), many Erlang uses weak cookie or place cookie in config file which can be use to join the cluster.

### Named Pipes

Named Pipes -> files stored in memory that get cleared out after being read

The Cobalt Strike's (uses Named Pipes for every command) workflow looks like following:

1. Beacon starts a named pipe of \\.\pipe\msagent\_12
2. Beacon starts a new process and injects command into that process directing output to \\.\pipe\msagent\_12
3. Server displays what was written into \\.\pipe\msagent\_12

Pipes are used between two applications or processes using SHM.

**Listing Named Pipes with Pipelist**

```cmd-session
C:\htb> pipelist.exe /accepteula

PipeList v1.02 - Lists open named pipes
Copyright (C) 2005-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Pipe Name                                    Instances       Max Instances
---------                                    ---------       -------------
InitShutdown                                      3               -1
lsass                                             4               -1
ntsvcs                                            3               -1
. . .
```

```powershell-session
PS C:\htb>  gci \\.\pipe\
```

**Reviewing LSASS Named Pipe Permissions**

```cmd-session
C:\htb> accesschk.exe /accepteula \\.\Pipe\lsass -v

Accesschk v6.12 - Reports effective permissions for securable objects
Copyright (C) 2006-2017 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\lsass
  Untrusted Mandatory Level [No-Write-Up]
  RW Everyone
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
. . .
  RW BUILTIN\Administrators
        FILE_ALL_ACCESS
```

from above only administrators have full access to the LSASS process (normal)

### Named Pipes Attack Example

{% embed url="https://www.exploit-db.com/exploits/48021" %}

<mark style="color:red;">search for all named pipes that allow write access</mark>

**Checking WindscribeService Named Pipe Permissions**

Confirming with `accesschk` we see that the Everyone group does indeed have `FILE_ALL_ACCESS` (All possible access rights) over the pipe.

```cmd-session
C:\htb> accesschk.exe -accepteula -w \pipe\WindscribeService -v

Accesschk v6.13 - Reports effective permissions for securable objects
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\WindscribeService
  Medium Mandatory Level (Default) [No-Write-Up]
  RW Everyone
        FILE_ALL_ACCESS
```

## Assessment

**What service is listening on 0.0.0.0:21? (two words)**

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption><p>FileZilla Server</p></figcaption></figure>

1. Get pid using `netstat -aof | findstr 0.0.0.0:21`&#x20;
2. Get taskname using `tasklist /fi "pid eq 1216"` (2320 is the pid got from step 1)

**Which account has WRITE\_DAC privileges over the \pipe\SQLLocal\SQLEXPRESS01 named pipe?**

<figure><img src="../../../.gitbook/assets/image (112).png" alt=""><figcaption><p>NT SERVICE\MSSQL$SQLEXPRESS01</p></figcaption></figure>

Make sure that the accesschk tool is installed on the system before using it.
