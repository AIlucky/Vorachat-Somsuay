# John The Ripper

### Cracking Modes

**Single Crack Mode**

```bash
john --format=<hash_type> <hash or hash_file>
```

eg.

```shell-session
john --format=sha256 hashes_to_crack.txt
```

> The output will be on the console and the file "john.pot" (`~/.john/john.pot`) to the current user's home directory

**Show password**

```
john --show
```

**Cracking with John**

<table data-header-hidden><thead><tr><th width="154.33333333333334"></th><th width="256"></th><th></th></tr></thead><tbody><tr><td><strong>Hash Format</strong></td><td><strong>Example Command</strong></td><td><strong>Description</strong></td></tr><tr><td>afs</td><td><code>john --format=afs hashes_to_crack.txt</code></td><td>AFS (Andrew File System) password hashes</td></tr><tr><td>bfegg</td><td><code>john --format=bfegg hashes_to_crack.txt</code></td><td>bfegg hashes used in Eggdrop IRC bots</td></tr><tr><td>bf</td><td><code>john --format=bf hashes_to_crack.txt</code></td><td>Blowfish-based crypt(3) hashes</td></tr><tr><td>bsdi</td><td><code>john --format=bsdi hashes_to_crack.txt</code></td><td>BSDi crypt(3) hashes</td></tr><tr><td>crypt(3)</td><td><code>john --format=crypt hashes_to_crack.txt</code></td><td>Traditional Unix crypt(3) hashes</td></tr><tr><td>des</td><td><code>john --format=des hashes_to_crack.txt</code></td><td>Traditional DES-based crypt(3) hashes</td></tr><tr><td>dmd5</td><td><code>john --format=dmd5 hashes_to_crack.txt</code></td><td>DMD5 (Dragonfly BSD MD5) password hashes</td></tr><tr><td>dominosec</td><td><code>john --format=dominosec hashes_to_crack.txt</code></td><td>IBM Lotus Domino 6/7 password hashes</td></tr><tr><td>EPiServer SID hashes</td><td><code>john --format=episerver hashes_to_crack.txt</code></td><td>EPiServer SID (Security Identifier) password hashes</td></tr><tr><td>hdaa</td><td><code>john --format=hdaa hashes_to_crack.txt</code></td><td>hdaa password hashes used in Openwall GNU/Linux</td></tr><tr><td>hmac-md5</td><td><code>john --format=hmac-md5 hashes_to_crack.txt</code></td><td>hmac-md5 password hashes</td></tr><tr><td>hmailserver</td><td><code>john --format=hmailserver hashes_to_crack.txt</code></td><td>hmailserver password hashes</td></tr><tr><td>ipb2</td><td><code>john --format=ipb2 hashes_to_crack.txt</code></td><td>Invision Power Board 2 password hashes</td></tr><tr><td>krb4</td><td><code>john --format=krb4 hashes_to_crack.txt</code></td><td>Kerberos 4 password hashes</td></tr><tr><td>krb5</td><td><code>john --format=krb5 hashes_to_crack.txt</code></td><td>Kerberos 5 password hashes</td></tr><tr><td>LM</td><td><code>john --format=LM hashes_to_crack.txt</code></td><td>LM (Lan Manager) password hashes</td></tr><tr><td>lotus5</td><td><code>john --format=lotus5 hashes_to_crack.txt</code></td><td>Lotus Notes/Domino 5 password hashes</td></tr><tr><td>md4-gen</td><td><code>john --format=md4-gen hashes_to_crack.txt</code></td><td>Generic MD4 password hashes</td></tr><tr><td>md5</td><td><code>john --format=md5 hashes_to_crack.txt</code></td><td>MD5 password hashes</td></tr><tr><td>md5-gen</td><td><code>john --format=md5-gen hashes_to_crack.txt</code></td><td>Generic MD5 password hashes</td></tr><tr><td>mscash</td><td><code>john --format=mscash hashes_to_crack.txt</code></td><td>MS Cache password hashes</td></tr><tr><td>mscash2</td><td><code>john --format=mscash2 hashes_to_crack.txt</code></td><td>MS Cache v2 password hashes</td></tr><tr><td>mschapv2</td><td><code>john --format=mschapv2 hashes_to_crack.txt</code></td><td>MS CHAP v2 password hashes</td></tr><tr><td>mskrb5</td><td><code>john --format=mskrb5 hashes_to_crack.txt</code></td><td>MS Kerberos 5 password hashes</td></tr><tr><td>mssql05</td><td><code>john --format=mssql05 hashes_to_crack.txt</code></td><td>MS SQL 2005 password hashes</td></tr><tr><td>mssql</td><td><code>john --format=mssql hashes_to_crack.txt</code></td><td>MS SQL password hashes</td></tr><tr><td>mysql-fast</td><td><code>john --format=mysql-fast hashes_to_crack.txt</code></td><td>MySQL fast password hashes</td></tr><tr><td>mysql</td><td><code>john --format=mysql hashes_to_crack.txt</code></td><td>MySQL password hashes</td></tr><tr><td>mysql-sha1</td><td><code>john --format=mysql-sha1 hashes_to_crack.txt</code></td><td>MySQL SHA1 password hashes</td></tr><tr><td>NETLM</td><td><code>john --format=netlm hashes_to_crack.txt</code></td><td>NETLM (NT LAN Manager) password hashes</td></tr><tr><td>NETLMv2</td><td><code>john --format=netlmv2 hashes_to_crack.txt</code></td><td>NETLMv2 (NT LAN Manager version 2) password hashes</td></tr><tr><td>NETNTLM</td><td><code>john --format=netntlm hashes_to_crack.txt</code></td><td>NETNTLM (NT LAN Manager) password hashes</td></tr><tr><td>NETNTLMv2</td><td><code>john --format=netntlmv2 hashes_to_crack.txt</code></td><td>NETNTLMv2 (NT LAN Manager version 2) password hashes</td></tr><tr><td>NEThalfLM</td><td><code>john --format=nethalflm hashes_to_crack.txt</code></td><td>NEThalfLM (NT LAN Manager) password hashes</td></tr><tr><td>md5ns</td><td><code>john --format=md5ns hashes_to_crack.txt</code></td><td>md5ns (MD5 namespace) password hashes</td></tr><tr><td>nsldap</td><td><code>john --format=nsldap hashes_to_crack.txt</code></td><td>nsldap (OpenLDAP SHA) password hashes</td></tr><tr><td>ssha</td><td><code>john --format=ssha hashes_to_crack.txt</code></td><td>ssha (Salted SHA) password hashes</td></tr><tr><td>NT</td><td><code>john --format=nt hashes_to_crack.txt</code></td><td>NT (Windows NT) password hashes</td></tr><tr><td>openssha</td><td><code>john --format=openssha hashes_to_crack.txt</code></td><td>OPENSSH private key password hashes</td></tr><tr><td>oracle11</td><td><code>john --format=oracle11 hashes_to_crack.txt</code></td><td>Oracle 11 password hashes</td></tr><tr><td>oracle</td><td><code>john --format=oracle hashes_to_crack.txt</code></td><td>Oracle password hashes</td></tr><tr><td>pdf</td><td><code>john --format=pdf hashes_to_crack.txt</code></td><td>PDF (Portable Document Format) password hashes</td></tr><tr><td>phpass-md5</td><td><code>john --format=phpass-md5 hashes_to_crack.txt</code></td><td>PHPass-MD5 (Portable PHP password hashing framework) password hashes</td></tr><tr><td>phps</td><td><code>john --format=phps hashes_to_crack.txt</code></td><td>PHPS password hashes</td></tr><tr><td>pix-md5</td><td><code>john --format=pix-md5 hashes_to_crack.txt</code></td><td>Cisco PIX MD5 password hashes</td></tr><tr><td>po</td><td><code>john --format=po hashes_to_crack.txt</code></td><td>Po (Sybase SQL Anywhere) password hashes</td></tr><tr><td>rar</td><td><code>john --format=rar hashes_to_crack.txt</code></td><td>RAR (WinRAR) password hashes</td></tr><tr><td>raw-md4</td><td><code>john --format=raw-md4 hashes_to_crack.txt</code></td><td>Raw MD4 password hashes</td></tr><tr><td>raw-md5</td><td><code>john --format=raw-md5 hashes_to_crack.txt</code></td><td>Raw MD5 password hashes</td></tr><tr><td>raw-md5-unicode</td><td><code>john --format=raw-md5-unicode hashes_to_crack.txt</code></td><td>Raw MD5 Unicode password hashes</td></tr><tr><td>raw-sha1</td><td><code>john --format=raw-sha1 hashes_to_crack.txt</code></td><td>Raw SHA1 password hashes</td></tr><tr><td>raw-sha224</td><td><code>john --format=raw-sha224 hashes_to_crack.txt</code></td><td>Raw SHA224 password hashes</td></tr><tr><td>raw-sha256</td><td><code>john --format=raw-sha256 hashes_to_crack.txt</code></td><td>Raw SHA256 password hashes</td></tr><tr><td>raw-sha384</td><td><code>john --format=raw-sha384 hashes_to_crack.txt</code></td><td>Raw SHA384 password hashes</td></tr><tr><td>raw-sha512</td><td><code>john --format=raw-sha512 hashes_to_crack.txt</code></td><td>Raw SHA512 password hashes</td></tr><tr><td>salted-sha</td><td><code>john --format=salted-sha hashes_to_crack.txt</code></td><td>Salted SHA password hashes</td></tr><tr><td>sapb</td><td><code>john --format=sapb hashes_to_crack.txt</code></td><td>SAP CODVN B (BCODE) password hashes</td></tr><tr><td>sapg</td><td><code>john --format=sapg hashes_to_crack.txt</code></td><td>SAP CODVN G (PASSCODE) password hashes</td></tr><tr><td>sha1-gen</td><td><code>john --format=sha1-gen hashes_to_crack.txt</code></td><td>Generic SHA1 password hashes</td></tr><tr><td>skey</td><td><code>john --format=skey hashes_to_crack.txt</code></td><td>S/Key (One-time password) hashes</td></tr><tr><td>ssh</td><td><code>john --format=ssh hashes_to_crack.txt</code></td><td>SSH (Secure Shell) password hashes</td></tr><tr><td>sybasease</td><td><code>john --format=sybasease hashes_to_crack.txt</code></td><td>Sybase ASE password hashes</td></tr><tr><td>xsha</td><td><code>john --format=xsha hashes_to_crack.txt</code></td><td>xsha (Extended SHA) password hashes</td></tr><tr><td>zip</td><td><code>john --format=zip hashes_to_crack.txt</code></td><td>ZIP (WinZip) password hashes</td></tr></tbody></table>

**Wordlist Mode**

```shell-session
john --wordlist=<wordlist_file> --rules <hash_file>
```

{% hint style="info" %}
Multiple wordlists can be specified by separating them with a comma.
{% endhint %}

**Incremental Mode**

Most time-consuming of all the John modes.

```shell-session
john --incremental <hash_file>
```

### Cracking Files

John can crack password protected or encrypted files but it need additional tools.

eg.

```shell-session
pdf2john server_doc.pdf > server_doc.hash
```

then

```shell-session
john --wordlist=<wordlist.txt> server_doc.hash
```

<table data-header-hidden><thead><tr><th width="266"></th><th></th></tr></thead><tbody><tr><td><strong>Tool</strong></td><td><strong>Description</strong></td></tr><tr><td><code>pdf2john</code></td><td>Converts PDF documents for John</td></tr><tr><td><code>ssh2john</code></td><td>Converts SSH private keys for John</td></tr><tr><td><code>mscash2john</code></td><td>Converts MS Cash hashes for John</td></tr><tr><td><code>keychain2john</code></td><td>Converts OS X keychain files for John</td></tr><tr><td><code>rar2john</code></td><td>Converts RAR archives for John</td></tr><tr><td><code>pfx2john</code></td><td>Converts PKCS#12 files for John</td></tr><tr><td><code>truecrypt_volume2john</code></td><td>Converts TrueCrypt volumes for John</td></tr><tr><td><code>keepass2john</code></td><td>Converts KeePass databases for John</td></tr><tr><td><code>vncpcap2john</code></td><td>Converts VNC PCAP files for John</td></tr><tr><td><code>putty2john</code></td><td>Converts PuTTY private keys for John</td></tr><tr><td><code>zip2john</code></td><td>Converts ZIP archives for John</td></tr><tr><td><code>hccap2john</code></td><td>Converts WPA/WPA2 handshake captures for John</td></tr><tr><td><code>office2john</code></td><td>Converts MS Office documents for John</td></tr><tr><td><code>wpa2john</code></td><td>Converts WPA/WPA2 handshakes for John</td></tr></tbody></table>

{% hint style="info" %}
More of the tools can be found by typing `locate 2john` to the terminal.
{% endhint %}
