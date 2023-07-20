---
description: Most used in the internet
---

# WordPress - Discovery & Enumeration

## Intro

* Open-source and can be used for multiple purposes
* Usually used to host blog and forums
* Since it's **highly customizable makes it vulnerable through 3rd party plugins**
* Usually runs on Apache with MySQL as back-end
* It has individual files that allow us to identify that application

## Discovery/Footprinting

quick way to identify is looking at /robots.txt file. For wordpress it will look like this.

```shell-session
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```

`/wp-admin` and `/wp-content` directories would be a dead giveaway that we are dealing with WordPress.

accessing `wp-admin` directory will redirect us to the `wp-login.php` page.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/wp-login2.png" alt=""><figcaption><p>wp-login.php</p></figcaption></figure>

* WordPress stores plugins in wp-content/plugins GO LOOK THERE
* Themes are stored in wp-content/themes directory

There are five types of users on a standard WordPress installation.

1. **Administrator**: This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code.
2. **Editor**: An editor can publish and manage posts, including the posts of other users.
3. **Author**: They can publish and manage their own posts.
4. **Contributor**: These users can write and manage their own posts but cannot publish them.
5. **Subscriber**: These are standard users who can browse posts and edit their profiles.

**Administrator** is usually sufficient to obtain **code execution** on the server. **Editors** and **authors** might have access to certain **vulnerable plugins**, which normal users don’t.

## Enumeration

page source will give us what CMS it's using along with some hints to themes, plugins, and even usernames (if author names are published with posts).

* Spend time manually browsing the site
* look at page source of every page and find wp-content directory, themes and plugins, and list all interesting points.

Below is example of getting themes (we can enumerate further for version)

```shell-session
curl -s http://blog.inlanefreight.local/ | grep themes

<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.local/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css' type='text/css' media='all' />
```

Getting plugins

```shell-session
curl -s http://blog.inlanefreight.local/ | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.8' id='validation-engine-en-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8' id='validation-engine-js'></script>
		<link rel='stylesheet' id='mm_frontend-css'  href='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/js/index.js?ver=5.4.2' id='contact-form-7-js'></script>
```

from enumeration we know it's using `mail-masta` and `contact-form-7` plugins

Next is enumerating versions by browsing to the plugins path

[http://blog.inlanefreight.local/wp-content/plugins/mail-masta/](http://blog.inlanefreight.local/wp-content/plugins/mail-masta/) -> shows directory listing and readme.txt file is present. The file contains the version of the plugin, which we can look for vulnerability specific to this version (it's LFI published in Aug 2021)

Checking source of another pages may give us more plugins to explore.

```shell-session
curl -s http://blog.inlanefreight.local/?p=1 | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<link rel='stylesheet' id='wpdiscuz-frontend-css-css'  href='http://blog.inlanefreight.local/wp-content/plugins/wpdiscuz/themes/default/style.css?ver=7.0.4' type='text/css' media='all' />
```

We got a plugin that was not found before `wpdiscuz` version 7.0.4 which has [this](https://www.exploit-db.com/exploits/49967) unauthenticated remote code execution vulnerability from Jun 2021

## Enumerating Users

default login page can be found /wp-login.php

**Invalid password**

<figure><img src="https://academy.hackthebox.com/storage/modules/113/valid_user.png" alt=""><figcaption><p>http://blog.inlanefreight.local/wp-login.php</p></figcaption></figure>

**Invalid username**

<figure><img src="https://academy.hackthebox.com/storage/modules/113/invalid_user.png" alt=""><figcaption><p>http://blog.inlanefreight.local/wp-login.php</p></figcaption></figure>

As seen, it is vulnerable to username enumeration.

## WPScan

an automated WordPress scanner and enumeration tool. It determines if the various themes and plugins used by a blog are outdated or vulnerable.

It is free for 75 requests perday which uses the api token to check the number of request. To get the token, access [WPVulnDB](https://wpvulndb.com/)

```shell-session
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```

The default number of threads used is `5`. However, this value can be changed using the `-t` flag.

This scan helped us confirm some of the things we uncovered from manual enumeration

* WordPress core version 5.8 (directory listing enabled)
* Transport Gravity is in use which is a child theme of Business Gravity
* uncovered another username (john)

However, it missed the wpDiscuz and Contact Form 7 plugins. which we got from manual enumeration. <mark style="color:red;">SO, ENUM MANUALLY TOO!! and combine the results</mark>

## Assessment

### Notes

* Themes found
  * business-gravity
    * transport-gravity 1.0.0
* Plugins
  * mail-masta v5.8
  * contact-form-7 v5.4.2
  * wpdiscuz
  * wp sitemap page

Enumerate the `blog.inlanefreight.local` host and find a flag.txt flag in an accessible directory.

* WPScan

```
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.24
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://blog.inlanefreight.local/ [10.129.42.195]
[+] Started: Wed Jul 12 00:56:53 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.inlanefreight.local/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://blog.inlanefreight.local/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.inlanefreight.local/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://blog.inlanefreight.local/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.8 identified (Insecure, released on 2021-07-20).
<SNIP>
```

<figure><img src="../../../.gitbook/assets/image (38) (2).png" alt=""><figcaption><p>0ptions_ind3xeS_ftw!</p></figcaption></figure>

Perform manual enumeration to discover another installed plugin. Submit the plugin name as the answer (3 words).

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption><p>wp-sitemap-page</p></figcaption></figure>

Find the version number of this plugin. (i.e., 4.5.2)

```
http://blog.inlanefreight.local/wp-content/plugins/wp-sitemap-page/readme.txt
```

<figure><img src="../../../.gitbook/assets/image (47) (2) (2).png" alt=""><figcaption><p>1.6.4</p></figcaption></figure>
