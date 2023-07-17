---
description: Root account provides full administrative level access to the operating system
---

# 🐧 Linux Privilege Escalation

### Enumeration

Several helper scripts (such as [LinEnum](https://github.com/rebootuser/LinEnum)) exist to assist with enumeration.it is important to check several key details.

* `OS Version` -> Know the distribution will give the idea of what tools to use and find public exploits
* `Kernel Version` -> Kernel exploits may cause system to crash, be careful when running them against production. Understand the exploit before running it.
* `Running Service` -> Know what services are running as root, misconfiguration or vulnerability on the service can lead to privilege escalation

**List Current Processes**

```shell-session
ps aux | grep root

root         1  1.3  0.1  37656  5664 ?        Ss   23:26   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    23:26   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    23:26   0:00 [ksoftirqd/0]
root         4  0.0  0.0      0     0 ?        S    23:26   0:00 [kworker/0:0]
root         5  0.0  0.0      0     0 ?        S<   23:26   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S    23:26   0:00 [kworker/u8:0]
root         7  0.0  0.0      0     0 ?        S    23:26   0:00 [rcu_sched]
root         8  0.0  0.0      0     0 ?        S    23:26   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        S    23:26   0:00 [migration/0]

<SNIP>
```

* `Installed Packages and Versions` -> Check for outdated packages, e.g. Screen is a common terminal multiplexer allows session on many windows. Screen version 4.05.00 has a privilege escalation vulnerability!
* `Logged in Users` -> Know who are logged into system and what they can do for lateral movement and privilege escalation paths.

**List Current Processes**

```shell-session
ps au

USER       		PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      		1256  0.0  0.1  65832  3364 tty1     Ss   23:26   0:00 /bin/login --
cliff.moore     1322  0.0  0.1  22600  5160 tty1     S    23:26   0:00 -bash
shared     		1367  0.0  0.1  22568  5116 pts/0    Ss   23:27   0:00 -bash
root      		1384  0.0  0.1  52700  3812 tty1     S    23:29   0:00 sudo su
root      		1385  0.0  0.1  52284  3448 tty1     S    23:29   0:00 su
root      		1386  0.0  0.1  21224  3764 tty1     S+   23:29   0:00 bash
shared     		1397  0.0  0.1  37364  3428 pts/0    R+   23:30   0:00 ps au
```

* `User Home Directories` -> is the users' home directory accessible? if yes, check for ssh keys or scripts and configs containing credentails. Also check for .bash\_history for interesting commands.
  * Gaining ssh keys will also give a stable shell so definately look for one. Check for ARP cache to see other hosts being accessed and cross-reference these against any useable SSH private keys.
* `Sudo Privileges` -> what commands can the user run as root or another user? often sudoer entries include `NOPASSWD`, meaning that the user can run the specified command without being prompted for a password. Simply run `sudo -l` to check
* `Configuration Files` -> check files with extensions such as .conf and .config for usernames and passwords and other secrets.
* `Readable Shadow File` -> Get the password hashes for all users who have password set, we can try to crack the hash if the user was using weak password.
* `Password Hashes in /etc/passwd` -> this file is readable by all users where somecases the password hash might be stored in here.
* `Cron Jobs` -> similar to Windows schedule tasks weak permissions and other misconfigs might be present in these files which may lead to privilege escalation.
* `Unmounted File Systems and Additional Drives` -> check for mounted drives and find sensitive info using the command `lsblk`
* `SETUID and SETGID Permissions` -> Binaries with theses permissions allows user to run command as root, many binaries can be exploited to get a root shell.
*   `Writeable Directories` -> These can be useful to download files and some cronjob may place files in there.

    ```shell-session
    find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
    ```
*   `Writeable Files` -> Scripts and config files that are writable, with minor modification we might gain further access. Scripts that run as root using cron jobs can be modified to append command.

    ```shell-session
    find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
    ```



