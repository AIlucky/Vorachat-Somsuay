# Drupal - Discovery & Enumeration

## Intro

* open-source CMS that is popular among companies and developers.
* Drupal is written in PHP and supports using MySQL or PostgreSQL for the backend
* SQLite can be used if there's no DBMS installed
* Drupal allows users to use themes and modules like WordPress

## Discovery/Footprinting

_<mark style="color:purple;">**CMS' are often "juicy" targets**</mark>_

Drupal website can be identified in several ways by

* the **header** or **footer** message `Powered by Drupal`
* the standard **Drupal logo**
* the presence of a **`CHANGELOG.txt` file** or **`README.txt` file**
* the page source, or clues in the **robots.txt file** such as references to `/node`

```shell-session
curl -s http://drupal.inlanefreight.local | grep Drupal

<meta name="Generator" content="Drupal 8 (https://www.drupal.org)" />
<span>Powered by <a href="https://www.drupal.org">Drupal</a></span>
```

Another way to identify Drupal CMS is through [nodes](https://www.drupal.org/docs/8/core/modules/node/about-nodes).&#x20;

Drupal indexes its content using nodes. A node can hold anything such as a blog post, poll, article, etc. The page URIs are usually of the form `/node/<nodeid>`.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/drupal_node.png" alt=""><figcaption><p>http://drupal.inlanefreight.local/node/1</p></figcaption></figure>

the blog post above is found to be at `/node/1`

> This representation is helpful in identifying a Drupal website when a custom theme is in use.

Drupal supports three **types of users** by default:

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.

## Enumeration

Newer installs of Drupal by default block access to the `CHANGELOG.txt` and `README.txt` files, so we may need to do further enumeration.

**enumerating the version number using the `CHANGELOG.txt` file.**

```
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""

Drupal 7.57, 2018-02-21
```

**enumerating the version number using the `CHANGELOG.txt` file. **<mark style="color:red;">**Latest Drupal Version**</mark>

```shell-session
curl -s http://drupal.inlanefreight.local/CHANGELOG.txt

<!DOCTYPE html><html><head><title>404 Not Found</title></head><body><h1>Not Found</h1><p>The requested URL "http://drupal.inlanefreight.local/CHANGELOG.txt" was not found on this server.</p></body></html>
```

**scaning with `droopescan`**

```shell-session
droopescan scan drupal -u http://drupal.inlanefreight.local

[+] Plugins found:                                                              
    php http://drupal.inlanefreight.local/modules/php/
        http://drupal.inlanefreight.local/modules/php/LICENSE.txt

[+] No themes found.

[+] Possible version(s):
    8.9.0
    8.9.1

[+] Possible interesting urls found:
    Default admin - http://drupal.inlanefreight.local/user/login

[+] Scan finished (0:03:19.199526 elapsed)
```

Got the version number 8.9.1

## Assessment

Identify the Drupal version number in use on http://drupal-qa.inlanefreight.local

```
droopescan scan drupal -u http://drupal.inlanefreight.local
[+] Plugins found: 
    captcha http://drupal.inlanefreight.local/modules/captcha/
        http://drupal.inlanefreight.local/modules/captcha/README.md
        http://drupal.inlanefreight.local/modules/captcha/LICENSE.txt 
    php http://drupal.inlanefreight.local/modules/php/
        http://drupal.inlanefreight.local/modules/php/LICENSE.txt
              
[+] No themes found.

[+] Possible version(s):
    8.9.0
    8.9.1

[+] Possible interesting urls found:
    Default admin - http://drupal.inlanefreight.local/user/login

[+] Scan finished (0:05:37.072595 elapsed)
```

```
droopescan scan drupal -u http://drupal-qa.inlanefreight.local 
[+] Plugins found:                                                              
    profile http://drupal-qa.inlanefreight.local/modules/profile/
    php http://drupal-qa.inlanefreight.local/modules/php/
    image http://drupal-qa.inlanefreight.local/modules/image/

[+] Themes found:
    seven http://drupal-qa.inlanefreight.local/themes/seven/
    garland http://drupal-qa.inlanefreight.local/themes/garland/

[+] Possible version(s):
    7.30

[+] Possible interesting urls found:
    Default changelog file - http://drupal-qa.inlanefreight.local/CHANGELOG.txt
    Default admin - http://drupal-qa.inlanefreight.local/user/login

[+] Scan finished (0:04:40.788912 elapsed)
```

