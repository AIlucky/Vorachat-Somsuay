# Internal Information Gathering

## Setting Up Pivoting - SSH

**SSH dynamic port forwarding**

```shell-session
ssh -D 8081 -i dmz01_key root@10.129.203.111
```

**Setup proxychains4**

```shell-session
grep socks4 /etc/proxychains4.conf 

#	 	socks4	192.168.1.49	1080
#       proxy types: http, socks4, socks5
socks4 	127.0.0.1 8081
```

**Run nmap scan through proxychains**

```shell-session
proxychains nmap -sT -p 21,22,80,8080 172.16.8.120
```

## Active Directory Quick Hits - SMB NULL SESSION

### Enum4linux

```shell-session
proxychains enum4linux -U -P 172.16.8.3
```











