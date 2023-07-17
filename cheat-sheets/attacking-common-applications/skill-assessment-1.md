# Skill Assessment 1

During a penetration test against the company Inlanefreight, you have performed extensive enumeration and found the network to be quite locked down and well-hardened. You come across one host of particular interest that may be your ticket to an initial foothold. Enumerate the target host for potentially vulnerable applications, obtain a foothold, and submit the contents of the flag.txt file to complete this portion of the skills assessment.

**What vulnerable application is running?**

```
nmap -sC -sV -Pn 10.129.201.89 --open 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-14 04:39 EDT
Nmap scan report for 10.129.201.89
Host is up (0.19s latency).
Not shown: 593 filtered tcp ports (no-response), 400 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_09-01-21  08:07AM       <DIR>          website_backup
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Freight Logistics, Inc
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: APPS-SKILLS1
|   NetBIOS_Domain_Name: APPS-SKILLS1
|   NetBIOS_Computer_Name: APPS-SKILLS1
|   DNS_Domain_Name: APPS-SKILLS1
|   DNS_Computer_Name: APPS-SKILLS1
|   Product_Version: 10.0.17763
|_  System_Time: 2023-07-14T08:39:52+00:00
| ssl-cert: Subject: commonName=APPS-SKILLS1
| Not valid before: 2023-07-13T08:37:53
|_Not valid after:  2024-01-12T08:37:53
|_ssl-date: 2023-07-14T08:39:59+00:00; 0s from scanner time.
8080/tcp open  http          Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/9.0.0.M1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-07-14T08:39:52
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.46 seconds
```

* Tomcat

**What port is this application running on?**

* 8080

**What version of the application is in use?**

*

    <figure><img src="../../.gitbook/assets/image (6) (2).png" alt=""><figcaption><p>9.0.0.M1</p></figcaption></figure>

**Exploit the application to obtain a shell and submit the contents of the flag.txt file on the Administrator desktop.**

```
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/cgi/FUZZ.bat

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.201.89:8080/cgi/FUZZ.bat
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 307ms]
    * FUZZ: cmd

:: Progress: [4614/4614] :: Job [1/1] :: 193 req/sec :: Duration: [0:00:26] :: Errors: 0 ::
```

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/jaiguptanick/CVE-2019-0232" %}

<figure><img src="../../.gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (9) (3).png" alt=""><figcaption><p>f55763d31a8f63ec935abd07aee5d3d0</p></figcaption></figure>
