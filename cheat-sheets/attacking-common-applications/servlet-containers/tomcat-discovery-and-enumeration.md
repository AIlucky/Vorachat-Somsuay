---
description: open-source web server that hosts applications written in Java
---

# Tomcat - Discovery & Enumeration

## Intro

* Designed to run Java Servlets and JSP scripts
* Widely used by frameworks (Spring) and tools (Gradle)
* Less apt to be exposed but still exists
* Excellent foothold into internal network
* Usuallly found in internal pentest
* Will be marked as High Value Targets

## Discovery/Footprinting

If found tomcat, it could be easy foothold to internal network. But first we have to confirm whether it is actually tomcat.

* check server header in HTTP response
* check invalid page error to see default error page.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/tomcat_invalid.png" alt=""><figcaption><p>http://app-dev.inlanefreight.local:8080/invalid</p></figcaption></figure>

* If error page is customised, check /docs page

```bash
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 

<html lang="en"><head><META http-equiv="Content-Type" content="text/html; charset=UTF-8"><link href="./images/docs-stylesheet.css" rel="stylesheet" type="text/css"><title>Apache Tomcat 9 (9.0.30) - Documentation Index</title><meta name="author" 

<SNIP>
```

**Folder structure of a Tomcat.**

```shell-session
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```

* bin -> stores script and binaries to start tomcat server
* conf  -> stores various config files
* tomcat-users.xml -> stores user credentails and roles
* lib -> holds JAR files for currect functioning of tomcat
* logs, temp -> stores temp log fiels
* webapps -> default webroot of Tomcat and hosts all apps
* work -> store data during runtime (cache)

**Structure under webapps folder**

```shell-session
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```

MOST IMPORTANT FILE -> WEB-INF/web.xml

* known as deployment descriptor
* Stores information about routes used by the application and classes handling the routes

All compiled classes used by app should be stored in WEB-INF/classes, the classes might contain important logic and sensitive information

Any vulnerability in these files can lead to total compromise of the website

* lib -> stores the libraries needed by that particular application
* jsp -> stores [Jakarta Server Pages (JSP)](https://en.wikipedia.org/wiki/Jakarta\_Server\_Pages), formerly known as `JavaServer Pages`

**Example of web.xml file**

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```

The above file contains

* a new servlet named `AdminServlet` mapped to the `com.inlanefreight.api.AdminServlet`
  * java uses dot notation so it will be `classes/com/inlanefreight/api/AdminServlet.class`
* a new servlet mapping is created to map requests to `/admin` with `AdminServlet`
  * sends any request received for /admin to AdminServlet.class to process

`web.xml` descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.

**Example of tomcat-users.xml file**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<SNIP>
  
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->

   
 <SNIP>
  
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />


</tomcat-users>
```

The above files shows

* what each of the roles provide access to
  * `manager-gui`, `manager-script`, `manager-jmx`, and `manager-status`
* a user `tomcat` with the password `tomcat` has the `manager-gui` role
* a second weak password `admin` is set for the user account `admin`

## Enumeration

Look for  /manager and /host-manager pages unless it has a known vulnerability

### Locating /manager and /host-manager using Gobuster

```shell-session
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://web01.inlanefreight.local:8180/
[+] Threads:        10
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/21 17:34:54 Starting gobuster
===============================================================
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
Progress: 49959 / 87665 (56.99%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/09/21 17:44:29 Finished
===============================================================
```

We may be able to either log in to one of these using weak credentials such as `tomcat:tomcat`, `admin:admin`, etc. If didn't work, try brute forcing. If successful we can upload WAR file contianing JSP web shell and get RCE on Tomcat server.

## Assessment

What version of Tomcat is running on the application located at http://web01.inlanefreight.local:8180?

<figure><img src="../../../.gitbook/assets/ภาพ (15).png" alt=""><figcaption><p>10.0.10</p></figcaption></figure>

What role does the admin user have in the configuration example?

* ```
  admin-gui
  ```































