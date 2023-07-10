# Assessment Medium

## Enumeration

### Rustscan

```
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack
53/tcp  open  domain  syn-ack
110/tcp open  pop3    syn-ack
995/tcp open  pop3s   syn-ack

```

### Nmap

```
nmap --script "pop3-capabilities or pop3-ntlm-info" -sV -p 110,995 10.129.201.127 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-29 06:19 EDT
Nmap scan report for 10.129.201.127
Host is up (0.20s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) UIDL CAPA RESP-CODES STLS TOP AUTH-RESP-CODE USER PIPELINING
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: UIDL CAPA SASL(PLAIN) USER RESP-CODES AUTH-RESP-CODE TOP PIPELINING

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.53 seconds


# After restarting the server we got more ports as a result


# Nmap 7.94 scan initiated Thu Jun 29 06:54:41 2023 as: nmap -p- -sC -sV -O -oN medlab 10.129.249.13
Nmap scan report for 10.129.249.13
Host is up (0.20s latency).
Not shown: 65529 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
53/tcp    open  domain   ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
110/tcp   open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: USER CAPA PIPELINING SASL(PLAIN) UIDL RESP-CODES STLS TOP AUTH-RESP-CODE
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
995/tcp   open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: TOP SASL(PLAIN) AUTH-RESP-CODE PIPELINING USER CAPA RESP-CODES UIDL
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
2121/tcp  open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (InlaneFTP) [10.129.249.13]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 ProFTPD Server (InlaneFTP) [10.129.249.13]
30021/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x   2 ftp      ftp          4096 Apr 18  2022 simon
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Internal FTP) [10.129.249.13]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 ProFTPD Server (Internal FTP) [10.129.249.13]

```

## DNS 53

```
dig axfr inlanefreight.htb @10.129.201.127 

; <<>> DiG 9.18.13-1-Debian <<>> axfr inlanefreight.htb @10.129.201.127
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.200.5
dc1.inlanefreight.htb.  604800  IN      A       10.129.100.10
dc2.inlanefreight.htb.  604800  IN      A       10.129.200.10
int-ftp.inlanefreight.htb. 604800 IN    A       127.0.0.1
int-nfs.inlanefreight.htb. 604800 IN    A       10.129.200.70
ns.inlanefreight.htb.   604800  IN      A       127.0.0.1
un.inlanefreight.htb.   604800  IN      A       10.129.200.142
ws1.inlanefreight.htb.  604800  IN      A       10.129.200.101
ws2.inlanefreight.htb.  604800  IN      A       10.129.200.102
wsus.inlanefreight.htb. 604800  IN      A       10.129.200.80
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 208 msec
;; SERVER: 10.129.201.127#53(10.129.201.127) (TCP)
;; WHEN: Thu Jun 29 06:21:16 EDT 2023
;; XFR size: 13 records (messages 1, bytes 372)

```

## FTP

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>simon:8Ns8j1b!23hs4921smHzwn</p></figcaption></figure>

* SSH and get the flag.



