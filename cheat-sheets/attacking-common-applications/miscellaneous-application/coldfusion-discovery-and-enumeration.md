---
description: >-
  a programming language and a web application development platform based on
  Java.
---

# ColdFusion - Discovery & Enumeration

* used to build dynamic and interactive web applications that can be connected to various APIs and databases (MySQL, Oracle, and Microsoft SQL Server)
* ColdFusion Markup Language (`CFML`) is the proprietary programming language used in ColdFusion to develop dynamic web applications.
* CFML includes tags and functions for database integration, web services, email management, and other common web development tasks

`cfquery` tag can execute SQL statements to retrieve data from a database:

```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfque
```

`cfloop` tag to iterate through the records retrieved from the database:

```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```

* CFML enables developers to create complex business logic using minimal code.
* ColdFusion supports other programming languages, such as JavaScript and Java, allowing developers to use their preferred programming language

<table data-header-hidden><thead><tr><th width="209"></th><th></th></tr></thead><tbody><tr><td><strong>Benefits</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Developing data-driven web applications</code></td><td>ColdFusion allows developers to build rich, responsive web applications easily. It offers session management, form handling, debugging, and more features. ColdFusion allows you to leverage your existing knowledge of the language and combines it with advanced features to help you build robust web applications quickly.</td></tr><tr><td><code>Integrating with databases</code></td><td>ColdFusion easily integrates with databases such as Oracle, SQL Server, and MySQL. ColdFusion provides advanced database connectivity and is designed to make it easy to retrieve, manipulate, and view data from a database and the web.</td></tr><tr><td><code>Simplifying web content management</code></td><td>One of the primary goals of ColdFusion is to streamline web content management. The platform offers dynamic HTML generation and simplifies form creation, URL rewriting, file uploading, and handling of large forms. Furthermore, ColdFusion also supports AJAX by automatically handling the serialisation and deserialisation of AJAX-enabled components.</td></tr><tr><td><code>Performance</code></td><td>ColdFusion is designed to be highly performant and is optimised for low latency and high throughput. It can handle a large number of simultaneous requests while maintaining a high level of performance.</td></tr><tr><td><code>Collaboration</code></td><td>ColdFusion offers features that allow developers to work together on projects in real-time. This includes code sharing, debugging, version control, and more. This allows for faster and more efficient development, reduced time-to-market and quicker delivery of projects.</td></tr></tbody></table>

vulnerabilities of ColdFusion:

1. CVE-2021-21087: Arbitrary disallow of uploading JSP source code
2. CVE-2020-24453: Active Directory integration misconfiguration
3. CVE-2020-24450: Command injection vulnerability
4. CVE-2020-24449: Arbitrary file reading vulnerability
5. CVE-2019-15909: Cross-Site Scripting (XSS) Vulnerability

ColdFusion exposes a fair few ports by default:

<table><thead><tr><th width="88.33333333333331">Port</th><th width="146">Protocol</th><th>Description</th></tr></thead><tbody><tr><td>80</td><td>HTTP</td><td>Used for non-secure HTTP communication between the web server and web browser.</td></tr><tr><td>443</td><td>HTTPS</td><td>Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.</td></tr><tr><td>1935</td><td>RPC</td><td>Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.</td></tr><tr><td>25</td><td>SMTP</td><td>Simple Mail Transfer Protocol (SMTP) is used for sending email messages.</td></tr><tr><td>8500</td><td>SSL</td><td>Used for server communication via Secure Socket Layer (SSL).</td></tr><tr><td>5500</td><td>Server Monitor</td><td>Used for remote administration of the ColdFusion server.</td></tr></tbody></table>

It's important to note that default ports can be changed during installation or configuration.

### Enumeration

```shell-session
nmap -p- -sC -Pn 10.129.247.30 --open

Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-13 11:45 GMT
Nmap scan report for 10.129.247.30
Host is up (0.028s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 350.38 seconds
```

Two Windows RPC services, and one running on `8500`. As we know, `8500` is a default port that ColdFusion uses for SSL. Navigating to the `IP:8500` lists 2 directories, `CFIDE` and `cfdocs,` in the root, further indicating that ColdFusion is running on port 8500.

The `/CFIDE/administrator` path, however, loads the ColdFusion 8 Administrator login page. Now we know for certain that `ColdFusion 8` is running on the server.

![](https://academy.hackthebox.com/storage/modules/113/coldfusion/CF8.png)

