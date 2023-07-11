# Bypassing filters

## Client-Side Validation

* The validation is done on client side (front end application)
* This could be bypassed by changing the request during transmission using proxy app (burp)
* We can upload the malicious file in different format first and then change the file name back
* Or change it at the UI of the application (search for `accept="" and onchange function` )

## Blacklist Filters

* Extension Black list
  * Try uploading with multiple extensions (ffuf, intruder) and check which works.
    * [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) and [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) for web extensions.
  * After finding what extensions work, try uploading a shell with that extension (choose the one that allows us to execute code.)

## Whitelist

* More secure than blacklist (only allow allowed extension)
* To bypass these try fuzzing the web application with various extensions
  * [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) and [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) for web extensions.
* Try double extensions from the [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) word list or other of tech is defferent.
* Try Null Byte
  * `%20`
  * `%0a`
  * `%00`
  * `%0d0a`
  * `/`
  * `.\`
  * `.`
  * `…`
  * `:`

## Type Filters

* **Content-Type** checks -> use a content type word list from seclists to fuzz the webapp and find what content type are available.
* MIME-Type checks -> basically it checks for magic bytes, the easiest way to bypass them is to add `GIF8` at the start and it will interpret as a gif file which will still be executable.

