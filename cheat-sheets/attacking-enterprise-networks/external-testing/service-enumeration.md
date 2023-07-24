# Service Enumeration

## FTP

```shell-session
ftp 10.129.203.101 # Connect to FTP server (anonymously) anonymous:anonymous

ftp> put test.txt # upload file
```

## SSH

```shell-session
nc -nv 10.129.203.101 22 # Banner grabing

ssh admin@10.129.203.101 # making a connection
```

## Email

```shell-session
sudo nmap -sV -sC -p25 10.129.203.101 # Port specific NMAP scan

telnet 10.129.203.101 25 # connecting to mail server
```

## Assessment

Enumerate the accessible services and find a flag. Submit the flag value as your answer (flag format: HTB{ }).

<details>

<summary>Enumeration</summary>

#### Nmap Scan

```
sudo nmap -sC -sV -A -oA services 10.129.58.85
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-23 04:55 EDT
Nmap scan report for 10.129.58.85
Host is up (0.30s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              38 May 30  2022 flag.txt
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp   open  domain   (unknown banner: 1337_HTB_DNS)
| dns-nsid: 
|_  bind.version: 1337_HTB_DNS
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    1337_HTB_DNS
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Inlanefreight
110/tcp  open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: STLS SASL CAPA UIDL RESP-CODES AUTH-RESP-CODE TOP PIPELINING
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
143/tcp  open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: Pre-login more listed SASL-IR have ID post-login LITERAL+ OK ENABLE LOGIN-REFERRALS capabilities LOGINDISABLEDA0001 IMAP4rev1 STARTTLS IDLE
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_imap-capabilities: Pre-login listed AUTH=PLAINA0001 more ID have LITERAL+ OK ENABLE LOGIN-REFERRALS post-login capabilities IMAP4rev1 IDLE SASL-IR
995/tcp  open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER CAPA UIDL RESP-CODES AUTH-RESP-CODE TOP PIPELINING
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Support Center
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.94%I=7%D=7/23%Time=64BCEAF9%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,39,"\x007\0\x06\x85\0\0\x01\0\x01\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\r\x0c1337_HTB_DNS");
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94%E=4%D=7/23%OT=21%CT=1%CU=30877%PV=Y%DS=2%DC=T%G=Y%TM=64BCEB3
OS:D%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=10D%TI=Z%CI=Z%TS=A)SEQ(SP=1
OS:08%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=9)SEQ(SP=108%GCD=1%ISR=10D%TI=Z%CI=Z%
OS:II=I%TS=A)OPS(O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11N
OS:W7%O5=M53CST11NW7%O6=M53CST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE8
OS:8%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40
OS:%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%
OS:W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=
OS:)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%
OS:DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: Host:  ubuntu; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   210.35 ms 10.10.14.1
2   267.05 ms 10.129.58.85

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.42 seconds
```

#### FTP anonymous login

```
ftp 10.129.58.85
Connected to 10.129.58.85.
220 (vsFTPd 3.0.3)
Name (10.129.58.85:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||40748|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              38 May 30  2022 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||46726|)
150 Opening BINARY mode data connection for flag.txt (38 bytes).
100% |*******************************|    38       15.07 KiB/s    00:00 ETA
226 Transfer complete.
38 bytes received in 00:00 (0.17 KiB/s)
ftp> !cat flag.txt
HTB{0eb0ab788df18c3115ac43b1c06ae6c4}
ftp>
```



</details>

* ```
  HTB{0eb0ab788df18c3115ac43b1c06ae6c4}
  ```
