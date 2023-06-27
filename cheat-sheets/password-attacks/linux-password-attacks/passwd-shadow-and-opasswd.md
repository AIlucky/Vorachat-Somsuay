# Passwd, Shadow & Opasswd

### Passwd File

**Passwd Format**

<table data-header-hidden><thead><tr><th width="143"></th><th width="108"></th><th width="89"></th><th width="93"></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><code>cry0l1t3:</code></td><td><code>x:</code></td><td><code>1000:</code></td><td><code>1000:</code></td><td><code>cry0l1t3,,,:</code></td><td><code>/home/cry0l1t3:</code></td><td><code>/bin/bash</code></td></tr><tr><td>Login name</td><td>Password info</td><td>UID</td><td>GUID</td><td>Full name/comments</td><td>Home directory</td><td>Shell</td></tr></tbody></table>

In some cases `/etc/passwd` file is writeable by mistake. Which will allow us to remove the `x` and the user will no longer have the password.

**Editing /etc/passwd - Before**

```shell-session
root:x:0:0:root:/root:/bin/bash
```

**Editing /etc/passwd - After**

```shell-session
root::0:0:root:/root:/bin/bash
```

Now we can just switch user (`su` + Enter). it will not ask for a password anymore and we will be able to switch to root.

### Shadow File

**Shadow Format**

<table data-header-hidden><thead><tr><th width="143"></th><th width="192"></th><th></th><th width="93"></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><code>cry0l1t3:</code></td><td><code>$6$wBRzy$...SNIP...x9cDWUxW1:</code></td><td><code>18937:</code></td><td><code>0:</code></td><td><code>99999:</code></td><td><code>7</code></td><td><code>:</code></td><td><code>:</code></td><td><code>:</code></td></tr><tr><td>Username</td><td>Encrypted password</td><td>Last PW change</td><td>Min. PW age</td><td>Max. PW age</td><td>Warning period</td><td>Inactivity period</td><td>Expiration date</td><td>Unused</td></tr></tbody></table>

The `encrypted password` has a format.

* `$<type>$<salt>$<hashed>`

The encrypted passwords are divided into three parts. The types of encryption allow us to distinguish between the following:

**Algorithm Types**

* `$1$` – MD5
* `$2a$` – Blowfish
* `$2y$` – Eksblowfish
* `$5$` – SHA-256
* `$6$` – SHA-512

By default, the SHA-512 (`$6$`) encryption method is used on the latest Linux distributions.

### Opasswd

`/etc/security/opasswd` is the file where old passwords are stored.

> Administrator/root permissions are also required to read the file if the permissions for this file have not been changed manually.

### Cracking Linux Credentials

**Unshadow**

```shell-session
sudo cp /etc/passwd /tmp/passwd.bak 
sudo cp /etc/shadow /tmp/shadow.bak 
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

**Hashcat - Cracking Unshadowed Hashes**

```shell-session
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

**Hashcat - Cracking MD5 Hashes**

```shell-session
cat md5-hashes.list
hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```

