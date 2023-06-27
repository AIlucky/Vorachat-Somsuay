# 🔑 Password Attacks

### Tips for OSCP

When there is a brute-force possibility in the test make sure to follow these

1. If it's an password brute-force make sure to use `rockyou.txt` if after a while it didn't work then cancle and it might not be a brute-force
2. For subdirectory enumeration use `/usr/share/wordlists/dirb/common...` if more results are needed then use `/usr/share/wordlists/dirbuster/directory_listing_medium...`

{% hint style="info" %}
some random guy on the reddit told to not use reddit cuz it's slow and has alot of duplicated credentials and if the target is suspected to be weak credentails try to use `/usr/share/seclists/Passwords/Default-Credentials` or `Common-Credentials` instead.
{% endhint %}

### Credential Storage

* Linux
  * The `shadow` file is located in `/etc/shadow` and is part of the Linux user management system.
  * In the past, the encrypted password was stored together with the username in the `/etc/passwd` file, but this was increasingly recognized as a security problem because the file can be viewed by all users on the system and must be readable. The `/etc/shadow` file can only be read by the user `root`.
* Windows
  * The [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) is a database file in Windows operating systems that stores users' passwords. It can be used to authenticate local and remote users.This file is located in `%SystemRoot%/system32/config/SAM.`
  * Credential Manager allows users to save the credentials they use to access network resources and websites. Saved credentials are stored based on user profiles in each user's `Credential Locker`. Credentials are encrypted and stored at the `C:\Users[Username]\AppData\Local\Microsoft[Vault/Credentials]\`
  * Each Domain Controller hosts a file called `NTDS.dit` that is synchronized across all Domain Controllers with the exception of [Read-Only Domain Controllers](https://docs.microsoft.com/en-us/windows/win32/ad/rodc-and-active-directory-schema). NTDS.dit is a database file that stores the data in Active Directory, including but not limited to:
    * User accounts (username & password hash)
    * Group accounts
    * Computer accounts
    * Group policy objects

### Encryption Technologies

<table data-header-hidden><thead><tr><th width="230"></th><th></th></tr></thead><tbody><tr><td><strong>Encryption Technology</strong></td><td><strong>Description</strong></td></tr><tr><td><code>UNIX crypt(3)</code></td><td>Crypt(3) is a traditional UNIX encryption system with a 56-bit key.</td></tr><tr><td><code>Traditional DES-based</code></td><td>DES-based encryption uses the Data Encryption Standard algorithm to encrypt data.</td></tr><tr><td><code>bigcrypt</code></td><td>Bigcrypt is an extension of traditional DES-based encryption. It uses a 128-bit key.</td></tr><tr><td><code>BSDI extended DES-based</code></td><td>BSDI extended DES-based encryption is an extension of the traditional DES-based encryption and uses a 168-bit key.</td></tr><tr><td><code>FreeBSD MD5-based</code> (Linux &#x26; Cisco)</td><td>FreeBSD MD5-based encryption uses the MD5 algorithm to encrypt data with a 128-bit key.</td></tr><tr><td><code>OpenBSD Blowfish-based</code></td><td>OpenBSD Blowfish-based encryption uses the Blowfish algorithm to encrypt data with a 448-bit key.</td></tr><tr><td><code>Kerberos/AFS</code></td><td>Kerberos and AFS are authentication systems that use encryption to ensure secure entity communication.</td></tr><tr><td><code>Windows LM</code></td><td>Windows LM encryption uses the Data Encryption Standard algorithm to encrypt data with a 56-bit key.</td></tr><tr><td><code>DES-based tripcodes</code></td><td>DES-based tripcodes are used to authenticate users based on the Data Encryption Standard algorithm.</td></tr><tr><td><code>SHA-crypt hashes</code></td><td>SHA-crypt hashes are used to encrypt data with a 256-bit key and are available in newer versions of Fedora and Ubuntu.</td></tr><tr><td><code>SHA-crypt</code> and <code>SUNMD5 hashes</code> (Solaris)</td><td>SHA-crypt and SUNMD5 hashes use the SHA-crypt and MD5 algorithms to encrypt data with a 256-bit key and are available in Solaris.</td></tr><tr><td><code>...</code></td><td>and many more.</td></tr></tbody></table>

### Mutating passwords

```shell-session
 hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

### Default Passwords

Look at [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet)[ ](https://github.com/ihebski/DefaultCreds-cheat-sheet)page to find default password for most of the services. and [this ](https://www.softwaretestinghelp.com/default-router-username-and-password-list/)page for default passwords for routers.













