---
description: Python tool which removes the need to configure proxychains
---

# SSH Pivoting with Sshuttle

This only works for pivoting over SSH and does not provide other options for pivoting over TOR or HTTPS proxy servers.

**Running sshuttle**

```shell-session
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v 
```

* `-r` -> to connect to the remote machine with a username and password

sshuttle creates an entry in our `iptables` to redirect all traffic to the 172.16.5.0/23 network through the pivot host.

**Traffic Routing through iptables Routes**

```shell-session
carbonlky@htb[/htb]$ nmap -v -sV -p3389 172.16.5.19 -A -Pn
```





"ubuntu" and password "HTB\_@cademy\_stdnt!"
