# ⬆ File Upload

File upload vulnerabilities are amongst the most common vulnerabilities found in web and mobile applications, as we can see in the latest [CVE Reports](https://www.cvedetails.com/vulnerability-list/cweid-434/vulnerabilities.html).

Worst file upload -> unauthenticated arbitrary file upload

Attack caused by arbitrary file upload -> gaining remote command execution (upload shell/revshell)

attacks we may be able to perform to exploit the file upload functionality

* Introducing other vulnerabilities like `XSS` or `XXE`.
* Causing a `Denial of Service (DoS)` on the back-end server.
* Overwriting critical system files and configurations.
* And many others.
