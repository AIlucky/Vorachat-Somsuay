---
description: deeper into the internals of the host operating system
---

# Linux Services & Internals Enumeration

* What services and applications are installed?
* What services are running?
* What sockets are in use?
* What users, admins, and groups exist on the system?
* Who is current logged in? What users recently logged in?
* What password policies, if any, are enforced on the host?
* Is the host joined to an Active Directory domain?
* What types of interesting information can we find in history, log, and backup files
* Which files have been modified recently and how often? Are there any interesting patterns in file modification that could indicate a cron job in use that we may be able to hijack?
* Current IP addressing information
* Anything interesting in the `/etc/hosts` file?
* Are there any interesting network connections to other systems in the internal network or even outside the network?
* What tools are installed on the system that we may be able to take advantage of? (Netcat, Perl, Python, Ruby, Nmap, tcpdump, gcc, etc.)
* Can we access the `bash_history` file for any users and can we uncover any thing interesting from their recorded command line history such as passwords?
* Are any Cron jobs running on the system that we may be able to hijack?

## Internals

**Network Interfaces**

```shell-session
ip a
```

check interfaces through which our target system can communicate.

**Hosts**

```shell-session
cat /etc/hosts
```

**User's Last Login**

```shell-session
lastlog

Username         Port     From             Latest
root                                       **Never logged in**
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
```

This can give us an idea of how widely used this system is which can open up the potential for more misconfigurations or "messy" directories or command histories.

**Logged In Users**

```shell-session
w

 12:27:21 up 1 day, 16:55,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
cliff.mo pts/0    10.10.14.16      Tue19   40:54m  0.02s  0.02s -bash
```

if anyone else is currently on the system with us

**Command History**

```shell-session
history
```

Sometimes we can also find special history files created by scripts or programs

**Finding History Files**

```shell-session
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null

-rw------- 1 htb-student htb-student 387 Nov 27 14:02 /home/htb-student/.bash_history
```

**Cron**

```shell-session
ls -la /etc/cron.daily/
```

Check for any cron jobs on the system. With misconfigurations such as **relative paths** or **weak permissions**, they can leverage to escalate privileges when the scheduled cron job runs.

**Proc**

```shell-session
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```

The [proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html) (`proc` / `procfs`) is a particular filesystem in Linux that contains information about system processes, hardware, and other system information.

## Services

**Installed Packages**

```shell-session
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```

If it is a slightly older Linux system, the likelihood increases that we can find installed packages that may already have at least one vulnerability.

**Sudo Version**

```shell-session
sudo -V
```

check if the `sudo` version installed on the system is vulnerable to any legacy or recent exploits.

**Binaries**

```shell-session
ls -l /bin /usr/bin/ /usr/sbin/
```

**GTFObins**

```shell-session
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```

[GTFObins](https://gtfobins.github.io) provides an excellent platform that includes a list of binaries that can potentially be exploited to escalate our privileges on the target system. With the next oneliner, we can compare the existing binaries with the ones from GTFObins to see which binaries we should investigate later.

**Trace System Calls**

```shell-session
strace ping -c1 10.129.112.20
```

To track and analyze system calls and signal processing. It allows us to follow the flow of a program and understand how it accesses system resources, processes signals, and receives and sends data from the operating system.

**Configuration Files**

```shell-session
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

Users can read almost all configuration files on a Linux operating system if the administrator has kept them the same.

**Scripts**

```shell-session
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```

The scripts are similar to the configuration files. Often administrators are lazy and convinced of network security and neglect the internal security of their systems. Scripts can have wrong privilege.

**Running Services by User**

```shell-session
ps aux | grep root
```

look at the process list to get information about which scripts or binaries are in use and by which user.

## Assessment

What is the latest Python version that is installed on the target?

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption><p>Not the correct answer</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>3.11</p></figcaption></figure>













