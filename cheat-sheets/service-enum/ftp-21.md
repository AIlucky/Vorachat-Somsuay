# FTP \[21]

### Usage

**Anonymous login**

```
ftp <ip>
```

> for anonymous login use anonymous:anonymous

**Download All Available Files**

```shell-session
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

**Download and Upload**

```shell-session
ftp> get testupload.txt 
ftp> put testupload.txt 
```

### Enumeration

```shell-session
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

### Service Interaction

**nc**

```shell-session
nc -nv 10.129.14.136 21
```

**telnet**

```shell-session
telnet 10.129.14.136 21
```

**openssl**

```shell-session
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

