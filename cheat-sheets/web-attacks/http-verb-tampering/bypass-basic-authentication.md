---
description: >-
  Insecure Web Server Configurations can allow us to bypass the HTTP Basic
  Authentication prompt on certain pages.
---

# Bypass Basic Authentication

## Identification

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_add.jpg" alt=""><figcaption><p>File manager example http://SERVER_IP:PORT/</p></figcaption></figure>

If we click reset we will be promted with below basic authentication.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_reset.jpg" alt=""><figcaption><p>Basic Authentication http://SERVER_IP:PORT/</p></figcaption></figure>

To Identify the verbtampering vulnerability try authenticating with any credential

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_unauthorized.jpg" alt=""><figcaption><p>http://SERVER_IP:PORT/admin/reset.php</p></figcaption></figure>

We can see that the above page is located under /admin/reset.php.

Next we have to Identify weather the /admin directory is restricted or /reset.php is. To do so, we can access the /admin to see if the login promt still shows, if yes then the entire directory is restricted.

## Exploit

We have to use proxy tool like Burp and examine the reqeust to the page. We will see that in this example the page uses GET request, we can try to change it to POST and see if it allows us. To change follow the image below.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_change_request.jpg" alt=""><figcaption><p>Chage GET to POST</p></figcaption></figure>

if we still get the same error message (401) we can try with different method like HEAD. First, check if HEAD is allowed on the root dir of page.

```shell-session
curl -i -X OPTIONS http://SERVER_IP:PORT/

HTTP/1.1 200 OK
Date: 
Server: Apache/2.4.41 (Ubuntu)
Allow: POST,OPTIONS,HEAD,GET
Content-Length: 0
Content-Type: httpd/unix-directory
```

We can see that POST,OPTIONS,HEAD, and GET are allowed.&#x20;

![HEAD\_request](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_verb\_tampering\_HEAD\_request.jpg)

If the error doesn't exists then the HEAD method was successful in bypassing the Basic Authentication.





