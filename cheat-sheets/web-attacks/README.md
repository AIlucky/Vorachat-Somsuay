# 🕷 Web Attacks

## HTTP Verb Tampering

* Attacks\&exploits web that uses HTTP verbs and methods (POST, GET, HEAD, DELETE, etc.)
* Exploite by sending unexpected methods
* Can be used to bypass authorization and security control
* Can exploit web server config by sending malicious HTTP request

## Insecure Direct Object References (IDOR)

* IDOR is one of most common web vulnerabilities
* Leads to unauthorized access to data (view data that attacker shouldn't)
* Bad access control in back-end causes IDOR.
* Web app stores user info sequentially leads to attacker getting other users' information

## XML External Entity (XXE) Injection

* Many web app process XML data
* If web app use old XML libraries it may be possible to send malicious XML data to disclose file on the back-end server
* The file obtained may contain sensitive info like password or source code of web app
* XXE can also be used to steal hosting server credentials,
* &#x20;whice leads to server compromise and RCE
