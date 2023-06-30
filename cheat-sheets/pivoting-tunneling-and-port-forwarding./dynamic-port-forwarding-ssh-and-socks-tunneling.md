---
description: >-
  Port forwarding is a technique that allows us to redirect a communication
  request from one port to another
---

# Dynamic Port Forwarding - SSH \&SOCKS Tunneling

## Local **Port Forwarding**

### **Executing the Local Port Forward**

```shell-session
ssh -L 1234:localhost:3306 Ubuntu@10.129.202.64
```

* \-L tells SSH server to forward all data we send via port 1234 -> localhost:3306

### **Confirming Port Forward with Nmap**

```shell-session
nmap -v -sV -p1234 localhost

Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:18 EST
. . . SNIP . . .

PORT     STATE SERVICE VERSION
1234/tcp open  mysql   MySQL 8.0.28-0ubuntu0.20.04.3

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.18 seconds
```

### **Forwarding Multiple Ports**

```shell-session
ssh -L 1234:localhost:3306 8080:localhost:80 ubuntu@10.129.202.64
```

> If you type `ifconfig` on the target host, you will find that this server has multiple NICs:

## Dynamic port forwarding

Usually when we attack we want to know what services is running, to do so, we try to run a nmap scan and enumeration the host. This is impossible from our attack host if it dosn't route to the network we are trying to scan.

To perform the scan we'll have to perform dynamic port forwarding and pivot the traffic via compromised host in the said network. We can do so by starting SOCKS listener in our attack host then configure to forward the traffic via SSH to the target network.

* SOCKS -> Socket Secure helps us communicate with server with firewall restrictions.

### **Enabling Dynamic Port Forwarding with SSH**

```shell-session
ssh -D 9050 ubuntu@10.129.202.64
```

* \-D Ask SSH server to enable dynamic port forawading.

After the SSH server allows it, we then need a tool that cat route any tool's traffic over the port 9050. `proxychains`is capable of redirecting TCP connections through TOR, SOCKS, and HTTP/HTTPS proxy servers and also allows us to chain multiple proxy servers together.

Using proxy chains gives us advantage of hiding our IP address since the receiving host will only see the IP of pivot host.

To use proxychains we have to edit the /etc/proxychains.conf.

```shell-session
tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

### **Using Nmap with Proxychains**

```shell-session
proxychains nmap -v -sn 172.16.5.1-200
```

{% hint style="danger" %}
<mark style="color:orange;">**we can only perform a**</mark><mark style="color:orange;">** **</mark><mark style="color:orange;">**`full TCP connect scan`**</mark><mark style="color:orange;">** **</mark><mark style="color:orange;">**over proxychains**</mark>
{% endhint %}

The reason is because proxychains can't understand incomplete packets. Using partial scans we will receive incorrect results. Also, `host-alive` checks may not work against Windows targets because the Windows Defender firewall blocks ICMP requests

> Scanning the entire subnet or range of network will take really longtime with full TCP connect scan.

### **Enumerating the Windows Target through Proxychains**

```shell-session
proxychains nmap -v -Pn -sT 172.16.5.19
```

### Using Metasploit with Proxychains

```shell-session
proxychains msfconsole
```

### **Using xfreerdp with Proxychains**

```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

## Assessment

* ```
  ssh -D 9050 ubuntu@10.129.202.64
  ```
* Edited proxychains file and enable listening at port 9050.

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption><p>Dynamic port-forwarding</p></figcaption></figure>

Apply the concepts taught in this section to pivot to the internal network and use RDP (credentials: victor:pass@123) to take control of the Windows target on 172.16.5.19. Submit the contents of Flag.txt located on the Desktop.

* Ran xfreerdp with proxychains. `proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123`

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption><p>xfreerdp with proxychains and found the flag.txt</p></figcaption></figure>
