---
description: >-
  it is also important to understand what pieces of information to look for and
  to be able to perform enumeration manually
---

# Environment Enumeration

## Gaining Situational Awareness

Gain a basic knowlege of what system did you get your foothold on.

* `whoami` - what user are we running as
* `id` - what groups does our user belong to?
* `hostname` - what is the server named. can we gather anything from the naming convention?
* `ifconfig` or `ip -a` - what subnet did we land in, does the host have additional NICs in other subnets?
* `sudo -l` - can our user run anything with sudo (as another user as root) without needing a password? This can sometimes be the easiest win and we can do something like `sudo su` and drop right into a root shell.

checking out what operating system and version we are dealing with.

```shell-session
cat /etc/os-release

NAME="Ubuntu"
VERSION="20.04.4 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.4 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

In this example the target is running Ubuntu 20.04.4 LTS, with this information we have to know what is the latest version of Ubuntu, in the example the system seems pretty up to date and have to available kernel exploits for this specific version.

**Checking PATH veriable**

```shell-session
echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

**Checking all environment variables for our current user**

```shell-session
env

SHELL=/bin/bash
PWD=/home/htb-student
LOGNAME=htb-student
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/htb-student
LANG=en_US.UTF-8

<SNIP>
```

**Check Kernel version**

```shell-session
uname -a
# OR
cat /proc/version

lscpu 
# gather some additional information about the host 
```

**What login shells exist on the server?**

```shell-session
cat /etc/shells
```

If the following are in place then stop wasting time to priv esc

* [Exec Shield](https://en.wikipedia.org/wiki/Exec\_Shield)
* [iptables](https://linux.die.net/man/8/iptables)
* [AppArmor](https://apparmor.net/)
* [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
* [Fail2ban](https://github.com/fail2ban/fail2ban)
* [Snort](https://www.snort.org/faq/what-is-snort)
* [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)

**Looking at the drives and any shares on the system**

```shell-session
lsblk
```

If we discover and can mount an additional drive or unmounted file system, we may find sensitive files, passwords, or backups that can be leveraged to escalate privileges.

`lpstat` can be used to find information about any printers attached to the system. (to look at sensitive files)

**Check for credentials in `fstab` for mounted drives**

```shell-session
cat /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-BdLsBLE4CvzJUgtkugkof4S0dZG7gWR8HCNOlRdLWoXVOba2tYUMzHfFQAP9ajul / ext4 defaults 0 0
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/20b1770d-a233-4780-900e-7c99bc974346 /boot ext4 defaults 0 0
```

**Check routing table**

```shell-session
route
```

In a domain environment we'll definitely want to check `/etc/resolv.conf` if the host is configured to use internal DNS we may be able to use this as a starting point to query the Active Directory environment.

**Check the arp table**

to see what other hosts the target has been communicating with.

```shell-session
arp -a
```

**Get list of users**

```shell-session
cat /etc/passwd | cut -f1 -d:

root
daemon
bin
sys

...SNIP...

mrb3n
lxd
bjones
administrator.ilfreight
backupsvc
cliff.moore
logger
shared
stacey.jenkins
htb-student
```

**Identifying different hash algorithms from the first hash blocks**

| **Algorithm** | **Hash**      |
| ------------- | ------------- |
| Salted MD5    | `$1$`...      |
| SHA-256       | `$5$`...      |
| SHA-512       | `$6$`...      |
| BCrypt        | `$2a$`...     |
| Scrypt        | `$7$`...      |
| Argon2        | `$argon2i$`.. |

**Check for which user have shell** and what version they have. e.g. Bash V 4.1 is vulnerable to shellshock.

```shell-session
grep "*sh$" /etc/passwd
```

Each user has a group and the group may have specific privilege. To check for groups and what users are in the group use

```shell-session
cat /etc/group
```

This lists all groups, to list members of specific group use&#x20;

```shell-session
getent group sudo

sudo:x:27:mrb3n
```

The low hanging fruit will be the config files which may contain sensitive information. Search all files with .conf and .config extension for passwords and other secrets.

If a password is found, use it with all users in the system! we might get lucky and gain access to another account.

In some senarios we may find a file system (Note that the file system can be mounted and unmounted by root privilege only) It is worth checking the mounted file system for any sensitive information.

**Mounted File Systems**

```shell-session
df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            1,9G     0  1,9G   0% /dev
```

> if unmounted the file system will no longer be accessible. To view unmounted file system try the below command.

**Unmounted File Systems**

```shell-session
cat /etc/fstab | grep -v "#" | column -t

UUID=5bf16727-fcdf-4205-906c-0620aa4a058f  /          ext4  errors=remount-ro  0  1
UUID=BE56-AAE0                             /boot/efi  vfat  umask=0077         0  1
/swapfile                                  none       swap  sw                 0  0
```

We could try to find any hidden files and directory

**Locate All Hidden Files**

```shell-session
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student

-rw-r--r-- 1 htb-student htb-student 3771 Nov 27 11:16 /home/htb-student/.bashrc
-rw-rw-r-- 1 htb-student htb-student 180 Nov 27 11:36 /home/htb-student/.wget-hsts
-rw------- 1 htb-student htb-student 387 Nov 27 14:02 /home/htb-student/.bash_history
-rw-r--r-- 1 htb-student htb-student 807 Nov 27 11:16 /home/htb-student/.profile
-rw-r--r-- 1 htb-student htb-student 0 Nov 27 11:31 /home/htb-student/.sudo_as_admin_successful
-rw-r--r-- 1 htb-student htb-student 220 Nov 27 11:16 /home/htb-student/.bash_logout
-rw-rw-r-- 1 htb-student htb-student 162 Nov 28 13:26 /home/htb-student/.notes
```

**All Hidden Directories**

```shell-session
find / -type d -name ".*" -ls 2>/dev/null

   684822      4 drwx------   3 htb-student htb-student     4096 Nov 28 12:32 /home/htb-student/.gnupg
   790793      4 drwx------   2 htb-student htb-student     4096 Okt 27 11:31 /home/htb-student/.ssh
   684804      4 drwx------  10 htb-student htb-student     4096 Okt 27 11:30 /home/htb-student/.cache
   790827      4 drwxrwxr-x   8 htb-student htb-student     4096 Okt 27 11:32 /home/htb-student/CVE-2021-3156/.git
   684796      4 drwx------  10 htb-student htb-student     4096 Okt 27 11:30 /home/htb-student/.config
   655426      4 drwxr-xr-x   3 htb-student htb-student     4096 Okt 27 11:19 /home/htb-student/.local
   524808      4 drwxr-xr-x   7 gdm         gdm             4096 Okt 27 11:19 /var/lib/gdm3/.cache
```

**Temporary Files**

```shell-session
ls -l /tmp /var/tmp /dev/shm

/dev/shm:
total 0

/tmp:
total 52
-rw------- 1 htb-student htb-student    0 Nov 28 12:32 config-err-v8LfEU
drwx------ 3 root        root        4096 Nov 28 12:37 snap.snap-store
drwx------ 2 htb-student htb-student 4096 Nov 28 12:32 ssh-OKlLKjlc98xh
```

* `/var/tmp` -> data retension is 30 days
* `/tmp` -> data retension is 10 days

## Assessment

Enumerate the Linux environment and look for interesting files that might contain sensitive data. Submit the flag as the answer.

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption><p>HTB{1nt3rn4l_5cr1p7_l34k}</p></figcaption></figure>

* Assuming that we know that the flag wil contain the name `'HTB{[^}]*}'`&#x20;
* redirecting stdout erros to /dev/null
* \-l -> --files-with-matches
* \-r -> --recursive
