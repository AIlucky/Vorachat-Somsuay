---
description: >-
  LDAP (Lightweight Directory Access Protocol) is a protocol used to access and
  manage directory information.
---

# Attacking LDAP

<table data-header-hidden><thead><tr><th width="194"></th><th></th></tr></thead><tbody><tr><td><strong>Functionality</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Efficient</code></td><td>Efficient and fast queries and connections to directory services, thanks to its lean query language and non-normalised data storage.</td></tr><tr><td><code>Global naming model</code></td><td>Supports multiple independent directories with a global naming model that ensures unique entries.</td></tr><tr><td><code>Extensible and flexible</code></td><td>This helps to meet future and local requirements by allowing custom attributes and schemas.</td></tr><tr><td><code>Compatibility</code></td><td>It is compatible with many software products and platforms as it runs over TCP/IP and SSL directly, and it is <code>platform-independent</code>, suitable for use in heterogeneous environments with various operating systems.</td></tr><tr><td><code>Authentication</code></td><td>It provides <code>authentication</code> mechanisms that enable users to <code>sign on once</code> and access multiple resources on the server securely.</td></tr></tbody></table>

However, it also suffers some significant issues:

<table><thead><tr><th width="154">Functionality</th><th>Description</th></tr></thead><tbody><tr><td><code>Compliance</code></td><td>Directory servers <code>must be LDAP compliant</code> for service to be deployed, which may <code>limit the choice</code> of vendors and products.</td></tr><tr><td><code>Complexity</code></td><td><code>Difficult to use and understand</code> for many developers and administrators, who may not know how to configure LDAP clients correctly or use it securely.</td></tr><tr><td><code>Encryption</code></td><td>LDAP <code>does not encrypt its traffic by default</code>, which exposes sensitive data to potential eavesdropping and tampering. LDAPS (LDAP over SSL) or StartTLS must be used to enable encryption.</td></tr><tr><td><code>Injection</code></td><td><code>Vulnerable to LDAP injection attacks</code>, where malicious users can manipulate LDAP queries and <code>gain unauthorised access</code> to data or resources. To prevent such attacks, input validation and output encoding must be implemented.</td></tr></tbody></table>

LDAP is `commonly used` for providing a `central location` for `accessing` and `managing` directory services.

LDAP enables organisations to store, manage, and secure this information in a standardised way

### LDAP Injection

The attacker can `inject malicious code` or `characters` into LDAP queries to alter the application's behaviour, `bypass security measures`, and `access sensitive data` stored in the LDAP directory.

To test for LDAP injection, you can use input values that contain `special characters or operators` that can change the query's meaning:

<table><thead><tr><th width="117">Input</th><th>Description</th></tr></thead><tbody><tr><td><code>*</code></td><td>An asterisk <code>*</code> can <code>match any number of characters</code>.</td></tr><tr><td><code>( )</code></td><td>Parentheses <code>( )</code> can <code>group expressions</code>.</td></tr><tr><td><code>|</code></td><td>A vertical bar <code>|</code> can perform <code>logical OR</code>.</td></tr><tr><td><code>&#x26;</code></td><td>An ampersand <code>&#x26;</code> can perform <code>logical AND</code>.</td></tr><tr><td><code>(cn=*)</code></td><td>Input values that try to bypass authentication or authorisation checks by injecting conditions that <code>always evaluate to true</code> can be used. For example, <code>(cn=*)</code> or <code>(objectClass=*)</code> can be used as input values for a username or password fields.</td></tr></tbody></table>

LDAP injection attacks are `similar to SQL injection attacks` but target the LDAP directory service instead of a database.

suppose an application uses the following LDAP query to authenticate users:

```php
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

`$username` and `$password` contain the user's login credentials. An attacker could inject the `*` character into the `$username` or `$password` field to modify the LDAP query and bypass authentication.&#x20;

* If \* in username then LDAP query will match any user account with any password.
  * access to the application with any password
* If \* in password then LDAP query would match any user account with any password that contains the injected string
  * access to the application with any username

### Enumeration

**nmap**

```shell-session
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 14:43 SAST
Nmap scan report for 10.129.204.229
Host is up (0.18s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Login
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 149.73 seconds

```

**Injection**

As `OpenLDAP` runs on the server, it is safe to assume that the web application running on port `80` uses LDAP for authentication.

Attempting to log in using a wildcard character (`*`) in the username and password fields grants access to the system, effectively `bypassing any authentication measures that had been implemented`. This is a `significant` security issue as it allows anyone with knowledge of the vulnerability to `gain unauthorised access` to the system and potentially sensitive data.











