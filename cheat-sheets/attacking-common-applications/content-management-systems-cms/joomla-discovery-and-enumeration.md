---
description: >-
  free and open-source CMS used for discussion forums, photo galleries,
  e-Commerce, user-based communities, and more
---

# Joomla - Discovery & Enumeration

## Intro

* Written in PHP and uses MySQL in the back-end
* Joomla also uses extensions and templates
* Joomla collects some anonymous [usage statistics](https://developer.joomla.org/about/stats.html)
* statistics data can be queried via their public [API](https://developer.joomla.org/about/stats/api.html).

## Discovery/Footprinting

**fingerprint Joomla** by looking at the page source

```shell-session
curl -s http://dev.inlanefreight.local/ | grep Joomla

	<meta name="generator" content="Joomla! - Open Source Content Management" />
<SNIP>
```

**robots.txt file for a Joomla site**

```
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# https://www.robotstxt.org/orig.html

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

fingerprint the Joomla version if the `README.txt` file is present.

```shell-session
curl -s http://dev.inlanefreight.local/README.txt | head -n 5

1- What is this?
	* This is a Joomla! installation/upgrade package to version 3.x
	* Joomla! Official site: https://www.joomla.org
	* Joomla! 3.9 version history - https://docs.joomla.org/Special:MyLanguage/Joomla_3.9_version_history
	* Detailed changes in the Changelog: https://github.com/joomla/joomla-cms/commits/staging
```

In certain Joomla installs, we may be able to fingerprint the version from JavaScript files in the `media/system/js/` directory or by browsing to `administrator/manifests/files/joomla.xml`.

```shell-session
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -

<?xml version="1.0" encoding="UTF-8"?>
<extension version="3.6" type="file" method="upgrade">
  <name>files_joomla</name>
  <author>Joomla! Project</author>
  <authorEmail>admin@joomla.org</authorEmail>
  <authorUrl>www.joomla.org</authorUrl>
  <copyright>(C) 2005 - 2019 Open Source Matters. All rights reserved</copyright>
  <license>GNU General Public License version 2 or later; see LICENSE.txt</license>
  <version>3.9.4</version>
  <creationDate>March 2019</creationDate>
  
 <SNIP>
```

`cache.xml` file can help to give us the approximate version located in`plugins/system/cache/cache.xml`.

## Enumeration

[droopescan](https://github.com/droope/droopescan) -> plugin-based scanner that works for SilverStripe, WordPress, and Drupal with limited functionality for Joomla and Moodle.

```shell-session
droopescan scan joomla --url http://dev.inlanefreight.local/

[+] Possible version(s):                                                        
    3.8.10
    3.8.11
    3.8.11-rc
    3.8.12
    3.8.12-rc
    3.8.13
    3.8.7
    3.8.7-rc
    3.8.8
    3.8.8-rc
    3.8.9
    3.8.9-rc

[+] Possible interesting urls found:
    Detailed version information. - http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml
    Login page. - http://dev.inlanefreight.local/administrator/
    License file. - http://dev.inlanefreight.local/LICENSE.txt
    Version attribute contains approx version - http://dev.inlanefreight.local/plugins/system/cache/cache.xml

[+] Scan finished (0:00:01.523369 elapsed)
```

* Not much info returned except version number

Run joomlascan or joomscan (joomlascan uses python2.7)

```shell-session
python2.7 joomlascan.py -u http://dev.inlanefreight.local

-------------------------------------------
      	     Joomla Scan                  
   Usage: python joomlascan.py <target>    
    Version 0.5beta - Database Entries 1233
         created by Andrea Draghetti       
-------------------------------------------
Robots file found: 	 	 > http://dev.inlanefreight.local/robots.txt
No Error Log found

Start scan...with 10 concurrent threads!
Component found: com_actionlogs	 > http://dev.inlanefreight.local/index.php?option=com_actionlogs
	 On the administrator components
Component found: com_admin	 > http://dev.inlanefreight.local/index.php?option=com_admin
	 On the administrator components
Component found: com_ajax	 > http://dev.inlanefreight.local/index.php?option=com_ajax
	 But possibly it is not active or protected
	 LICENSE file found 	 > http://dev.inlanefreight.local/administrator/components/com_actionlogs/actionlogs.xml
	 LICENSE file found 	 > http://dev.inlanefreight.local/administrator/components/com_admin/admin.xml
	 LICENSE file found 	 > http://dev.inlanefreight.local/administrator/components/com_ajax/ajax.xml
	 Explorable Directory 	 > http://dev.inlanefreight.local/components/com_actionlogs/
	 Explorable Directory 	 > http://dev.inlanefreight.local/administrator/components/com_actionlogs/
	 Explorable Directory 	 > http://dev.inlanefreight.local/components/com_admin/
	 Explorable Directory 	 > http://dev.inlanefreight.local/administrator/components/com_admin/
Component found: com_banners	 > http://dev.inlanefreight.local/index.php?option=com_banners
	 But possibly it is not active or protected
	 Explorable Directory 	 > http://dev.inlanefreight.local/components/com_ajax/
	 Explorable Directory 	 > http://dev.inlanefreight.local/administrator/components/com_ajax/
	 LICENSE file found 	 > http://dev.inlanefreight.local/administrator/components/com_banners/banners.xml


<SNIP>
```

* Not as valuable as droopescan but gave us accessible directories and files
* admin portal is at http://dev.inlanefreight.local/administrator/index.php

Default admin in Joomla is `admin` password is set during installation.

* We can brute force the password using this [script](https://github.com/ajnik/joomla-bruteforce)

```shell-session
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
 
admin:admin
```

## Assessment

**Fingerprint the Joomla version in use on http://app.inlanefreight.local (Format: x.x.x)**

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption><p>3.10.0</p></figcaption></figure>

**Find the password for the admin user on http://app.inlanefreight.local**

```
    ____  _____  _____  __  __  ___   ___    __    _  _ 
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  ( 
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
(1337.today)
   
    --=[OWASP JoomScan
    +---++---==[Version : 0.0.7
    +---++---==[Update Date : [2018/09/23]
    +---++---==[Authors : Mohammad Reza Espargham , Ali Razmjoo
    --=[Code name : Self Challenge
    @OWASP_JoomScan , @rezesp , @Ali_Razmjo0 , @OWASP

Processing http://app.inlanefreight.local ...

[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.10.0

[+] Core Joomla Vulnerability
[++] PHPMailer Remote Code Execution Vulnerability
CVE : CVE-2016-10033
https://www.rapid7.com/db/modules/exploit/multi/http/phpmailer_arg_injection
https://github.com/opsxcq/exploit-CVE-2016-10033
EDB : https://www.exploit-db.com/exploits/40969/

PPHPMailer Incomplete Fix Remote Code Execution Vulnerability
CVE : CVE-2016-10045
https://www.rapid7.com/db/modules/exploit/multi/http/phpmailer_arg_injection
EDB : https://www.exploit-db.com/exploits/40969/

[+] Checking Directory Listing
[++] directory has directory listing : 
http://app.inlanefreight.local/administrator/components
http://app.inlanefreight.local/administrator/modules
http://app.inlanefreight.local/administrator/templates
http://app.inlanefreight.local/images/banners    
 
  
[+] Checking apache info/status files     
[++] Readable info/status files are not found    
 
[+] admin finder  
[++] Admin page : http://app.inlanefreight.local/administrator/   
 
[+] Checking robots.txt existing
[++] robots.txt is found 
path : http://app.inlanefreight.local/robots.txt 
 
Interesting path found from robots.txt    
http://app.inlanefreight.local/joomla/administrator/    
http://app.inlanefreight.local/administrator/
http://app.inlanefreight.local/bin/
http://app.inlanefreight.local/cache/     
http://app.inlanefreight.local/cli/
http://app.inlanefreight.local/components/
http://app.inlanefreight.local/includes/  
http://app.inlanefreight.local/installation/     
http://app.inlanefreight.local/language/  
http://app.inlanefreight.local/layouts/   
http://app.inlanefreight.local/libraries/ 
http://app.inlanefreight.local/logs/      
http://app.inlanefreight.local/modules/   
http://app.inlanefreight.local/plugins/   
http://app.inlanefreight.local/tmp/

[+] Finding common backup files name
```

```
sudo ./joomla-bruteforce/joomla-brute.py -u http://app.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
admin:turnkey
```











