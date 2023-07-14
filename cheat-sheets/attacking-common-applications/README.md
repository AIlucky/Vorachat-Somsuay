---
description: STUDY Delivery box by IppSec
---

# ⚙ Attacking Common Applications

## Introduction

There are wide variety of web application such as Content Management Systems (CMS), custom web applications, intranet portals used by developers and sysadmins, code repositories, network monitoring tools, ticketing systems, wikis, knowledge bases, issue trackers, servlet container applications, and more.

**One may not be vulnerable but other might.**

* All web app can suffer from same kinds of vulnerabilities and misconfigs [OWASP Top 10](https://owasp.org/www-project-top-ten/)
* Web app are becoming more attractive target for attacker and pentester
* Companies are transitioning to remote work and exposing applications
* Application can be foothold into internal environment

### Application Data

<table data-header-hidden><thead><tr><th width="329"></th><th></th></tr></thead><tbody><tr><td><strong>Category</strong></td><td><strong>Applications</strong></td></tr><tr><td><a href="https://enlyft.com/tech/web-content-management">Web Content Management</a></td><td>Joomla, Drupal, WordPress, DotNetNuke, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/application-servers">Application Servers</a></td><td>Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/security-information-and-event-management-siem">Security Information and Event Management (SIEM)</a></td><td>Splunk, Trustwave, LogRhythm, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/network-management">Network Management</a></td><td>PRTG Network Monitor, ManageEngine Opmanger, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/it-management-software">IT Management</a></td><td>Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/software-frameworks">Software Frameworks</a></td><td>JBoss, Axis2, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/customer-service-management">Customer Service Management</a></td><td>osTicket, Zendesk, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/search-engines">Search Engines</a></td><td>Elasticsearch, Apache Solr, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/software-configuration-management">Software Configuration Management</a></td><td>Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/software-development-tools">Software Development Tools</a></td><td>Jenkins, Atlassian Confluence, phpMyAdmin, etc.</td></tr><tr><td><a href="https://enlyft.com/tech/enterprise-application-integration">Enterprise Application Integration</a></td><td>Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc.</td></tr></tbody></table>

Many of these suffer from publicly known exploits or have functionality that can be abused to gain remote code execution, steal credentials, or access sensitive information with or without valid credentials.

### Common Applications

<table><thead><tr><th width="137">Application</th><th>Description</th></tr></thead><tbody><tr><td>WordPress</td><td><a href="https://wordpress.org/">WordPress</a> is an open-source Content Management System (CMS) that can be used for multiple purposes. It's often used to host blogs and forums. WordPress is highly customizable as well as SEO friendly, which makes it popular among companies. However, its customizability and extensible nature make it prone to vulnerabilities through third-party themes and plugins. WordPress is written in PHP and usually runs on Apache with MySQL as the backend.</td></tr><tr><td>Drupal</td><td><a href="https://www.drupal.org/">Drupal</a> is another open-source CMS that is popular among companies and developers. Drupal is written in PHP and supports using MySQL or PostgreSQL for the backend. Additionally, SQLite can be used if there's no DBMS installed. Like WordPress, Drupal allows users to enhance their websites through the use of themes and modules.</td></tr><tr><td>Joomla</td><td><a href="https://www.joomla.org/">Joomla</a> is yet another open-source CMS written in PHP that typically uses MySQL but can be made to run with PostgreSQL or SQLite. Joomla can be used for blogs, discussion forums, e-commerce, and more. Joomla can be customized heavily with themes and extensions and is estimated to be the third most used CMS on the internet after WordPress and Shopify.</td></tr><tr><td>Tomcat</td><td><a href="https://tomcat.apache.org/">Apache Tomcat</a> is an open-source web server that hosts applications written in Java. Tomcat was initially designed to run Java Servlets and Java Server Pages (JSP) scripts. However, its popularity increased with Java-based frameworks and is now widely used by frameworks such as Spring and tools such as Gradle.</td></tr><tr><td>Jenkins</td><td><a href="https://jenkins.io/">Jenkins</a> is an open-source automation server written in Java that helps developers build and test their software projects continuously. It is a server-based system that runs in servlet containers such as Tomcat. Over the years, researchers have uncovered various vulnerabilities in Jenkins, including some that allow for remote code execution without requiring authentication.</td></tr><tr><td>Splunk</td><td>Splunk is a log analytics tool used to gather, analyze and visualize data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics. Splunk deployments are often used to house sensitive data and could provide a wealth of information for an attacker if compromised. Historically, Splunk has not suffered from a considerable amount of known vulnerabilities aside from an information disclosure vulnerability (<a href="https://nvd.nist.gov/vuln/detail/CVE-2018-11409">CVE-2018-11409</a>), and an authenticated remote code execution vulnerability in very old versions (<a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4642">CVE-2011-4642</a>).</td></tr><tr><td>PRTG Network Monitor</td><td><a href="https://www.paessler.com/prtg">PRTG Network Monitor</a> is an agentless network monitoring system that can be used to monitor metrics such as uptime, bandwidth usage, and more from a variety of devices such as routers, switches, servers, etc. It utilizes an auto-discovery mode to scan a network and then leverages protocols such as ICMP, WMI, SNMP, and NetFlow to communicate with and gather data from discovered devices. PRTG is written in <a href="https://en.wikipedia.org/wiki/Delphi_(software)">Delphi</a>.</td></tr><tr><td>osTicket</td><td><a href="https://osticket.com/">osTicket</a> is a widely-used open-source support ticketing system. It can be used to manage customer service tickets received via email, phone, and the web interface. osTicket is written in PHP and can run on Apache or IIS with MySQL as the backend.</td></tr><tr><td>GitLab</td><td><a href="https://about.gitlab.com/">GitLab</a> is an open-source software development platform with a Git repository manager, version control, issue tracking, code review, continuous integration and deployment, and more. It was originally written in Ruby but now utilizes Ruby on Rails, Go, and Vue.js. GitLab offers both community (free) and enterprises versions of the software.</td></tr></tbody></table>

```shell-session
IP=10.129.42.195
$ printf "%s\t%s\n\n" "$IP" "app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local" | sudo tee -a /etc/hosts
```

## Application Discovery & Enumeration

* Many oranizations don't know everything on network and have very little visibility
* Pentest can help to enumerate and identify applications that have been forgotten
* Strong pentest skill will help a ton

**Nmap - Web Discovery**

```shell-session
nmap -p 80,443,8000,8080,8180,8888,1000 --open -oA web_discovery -iL scope_list
```

* [EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness) and [Aquatone](https://github.com/michenriksen/aquatone) can be fed raw Nmap XML scan output then inspect all hosts running web applications and take screenshots of each
* Screenshots can help us narrow down and build a more targeted list to save time

### Notes skeleton

`External Penetration Test - <Client Name>`

* `Scope` (In-scope IP addresses/ranges, URLs, any fragile hosts, testing timeframes, and any limitations or other relative information we need handy)
* `Client Points of Contact`
* `Credentials`
* `Discovery/Enumeration`
  * `Scans`
  * `Live hosts`
* `Application Discovery`
  * `Scans`
  * `Interesting/Notable Hosts`
* `Exploitation`
  * `<Hostname or IP>`
  * `<Hostname or IP>`
* `Post-Exploitation`
  * `<Hostname or IP>`
  * `<<Hostname or IP>`

### Initial Enumeration

below example is the scope that client provided us.

```bash
cat scope_list

app.inlanefreight.local
dev.inlanefreight.local
drupal-dev.inlanefreight.local
drupal-qa.inlanefreight.local
drupal-acc.inlanefreight.local
drupal.inlanefreight.local
blog-dev.inlanefreight.local
blog.inlanefreight.local
app-dev.inlanefreight.local
jenkins-dev.inlanefreight.local
jenkins.inlanefreight.local
web01.inlanefreight.local
gitlab-dev.inlanefreight.local
gitlab.inlanefreight.local
support-dev.inlanefreight.local
support.inlanefreight.local
inlanefreight.local
10.129.201.50
```

First, perform nmap scan of common web ports `80,443,8000,8080,8180,8888,10000` then run EyeWitness or Aquatone. (we can run another nmap scan with top 10k ports or all TCP ports)

```shell-session
sudo  nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list 
```

The output will be something like this

```shell-session
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-07 21:49 EDT
Stats: 0:00:07 elapsed; 1 hosts completed (4 up), 4 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 81.24% done; ETC: 21:49 (0:00:01 remaining)

Nmap scan report for app.inlanefreight.local (10.129.42.195)
Host is up (0.12s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for app-dev.inlanefreight.local (10.129.201.58)
Host is up (0.12s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt
8009/tcp open  ajp13
8080/tcp open  http-proxy
8180/tcp open  unknown
8888/tcp open  sun-answerbook

Nmap scan report for gitlab-dev.inlanefreight.local (10.129.201.88)
Host is up (0.12s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8081/tcp open  blackice-icecap

Nmap scan report for 10.129.201.50
Host is up (0.13s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
8000/tcp open  http-alt
8080/tcp open  http-proxy
8089/tcp open  unknown

<SNIP>
```

We may identify several hosts

* Pay close attension to hostname
* Hosts with `dev` in FQDN are worth noting down

From the above scan `gitlab-dev.inlanefreight.local` seems interesting as it contains the word dev and git is interesting by itself.&#x20;

We may be able to access Git repos that contains sensitive information like credentials or other info like hidden subdomains/Vhosts. It is worth checking previous commits for credentials or other data.

**Further enumeration**

```shell-session
sudo nmap --open -sV 10.129.201.50

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-07 21:58 EDT
Nmap scan report for 10.129.201.50
Host is up (0.13s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp open  http          Splunkd httpd
8080/tcp open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd (free license; remote login disabled)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.63 seconds
```

From the above scan

* 80 : running IIS web server
* 8000/8089 : running Splunk
* 8080 : running PRTG Network Monitor

### Using EyeWitness

```shell-session
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

* \--web -> take screenshots using the Nmap XML output from the discovery scan as input.

### Using Aquatone

```shell-session
cat web_discovery.xml | ./aquatone -nmap
```

similar to EyeWitness, aquatone uses nmap's xml output to perform the enumeration.

### Interpreting the Results

![example report from the recon tool above](https://academy.hackthebox.com/storage/modules/113/eyewitness4.png)

As seen we found Tomcat where we can try default credentials on the `/manager` and `/host-manager` endpoints. If we can access either, we can upload a malicious WAR file and achieve remote code execution on the underlying host using [JSP code](https://en.wikipedia.org/wiki/Jakarta\_Server\_Pages).

Next is to look whether it is running CMS or custom web application.&#x20;

From the image below we can see that inlanefreight.local is running custom webapplication. And support-dev.inlanefreight.local appears to be running osTicket which has several vulnerabilities in the past.

Support ticketing system is interesting by itself because we may login and gain access to sensitive information.

![image](https://academy.hackthebox.com/storage/modules/113/eyewitness3.png)

{% hint style="danger" %}
We should not get careless and begin attacking hosts right away, as we may end up down a rabbit hole and miss something crucial later in the report
{% endhint %}

## Assessment

**vHosts needed for these questions:**

* app.inlanefreight.local
* dev.inlanefreight.local
* drupal-dev.inlanefreight.local
* drupal-qa.inlanefreight.local
* drupal-acc.inlanefreight.local
* drupal.inlanefreight.local
* blog.inlanefreight.local

## Application-Specific Hardening Tips

Though the general concepts for application hardening apply to all applications that we discussed in this module and will encounter in the real world, we can take some more specific measures. Here are a few:

<table><thead><tr><th width="160.33333333333331">Application</th><th width="202">Hardening Category</th><th>Discussion</th></tr></thead><tbody><tr><td><a href="https://wordpress.org/support/article/hardening-wordpress/">WordPress</a></td><td>Security monitoring</td><td>Use a security plugin such as <a href="https://www.wordfence.com/">WordFence</a> which includes security monitoring, blocking of suspicious activity, country blocking, two-factor authentication, and more</td></tr><tr><td><a href="https://docs.joomla.org/Security_Checklist/Joomla!_Setup">Joomla</a></td><td>Access controls</td><td>A plugin such as <a href="https://extensions.joomla.org/extension/adminexile/">AdminExile</a> can be used to require a secret key to log in to the Joomla admin page such as <code>http://joomla.inlanefreight.local/administrator?thisismysecretkey</code></td></tr><tr><td><a href="https://www.drupal.org/docs/security-in-drupal">Drupal</a></td><td>Access controls</td><td>Disable, hide, or move the <a href="https://www.drupal.org/docs/7/managing-users/hide-user-login">admin login page</a></td></tr><tr><td><a href="https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html">Tomcat</a></td><td>Access controls</td><td>Limit access to the Tomcat Manager and Host-Manager applications to only localhost. If these must be exposed externally, enforce IP whitelisting and set a very strong password and non-standard username.</td></tr><tr><td><a href="https://www.jenkins.io/doc/book/security/securing-jenkins/">Jenkins</a></td><td>Access controls</td><td>Configure permissions using the <a href="https://plugins.jenkins.io/matrix-auth">Matrix Authorization Strategy plugin</a></td></tr><tr><td><a href="https://docs.splunk.com/Documentation/Splunk/8.2.2/Security/Hardeningstandards">Splunk</a></td><td>Regular updates</td><td>Make sure to change the default password and ensure that Splunk is properly licensed to enforce authentication</td></tr><tr><td><a href="https://kb.paessler.com/en/topic/61108-what-security-features-does-prtg-include">PRTG Network Monitor</a></td><td>Secure authentication</td><td>Make sure to stay up-to-date and change the default PRTG password</td></tr><tr><td>osTicket</td><td>Access controls</td><td>Limit access from the internet if possible</td></tr><tr><td><a href="https://about.gitlab.com/blog/2020/05/20/gitlab-instance-security-best-practices/">GitLab</a></td><td>Secure authentication</td><td>Enforce sign-up restrictions such as requiring admin approval for new sign-ups, configuring allowed and denied domains</td></tr></tbody></table>
