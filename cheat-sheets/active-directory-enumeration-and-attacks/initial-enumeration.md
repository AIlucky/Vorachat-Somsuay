# Initial Enumeration

## Key Data Points

* AD Users
  * Try to enumerate valid usernames then we can target them for password spraying attacks.
* AD Joined Computers
  * This could be a Domain Controller, file server, Database server, web server, Email server, etc.
* Key Services
  * Kerberos, NetBIOS,  LDAP, DNS
* Vulnerable Hosts and Services.
  * Just any service or hosts that could easily be exploited and gain foothold.

## TTPs

Starting with passively identifying any hosts on the network, then followed by active validation of the results (to find services, names, potential vunls, etc.). Once hosts are found, we can find more info on those hosts. After a while we should map out all the info we have, we might have a set of credentials or accounts to target for a foothold to a domain joined host, or maybe resources to begin credentialed enumeration from our attack host.

### Identifying Hosts

Wireshark and TCPDump are useful to capture network traffic and gain types of network traffic.

#### **Wireshark**

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>ARP gives us the hosts IP.</p></figcaption></figure>

#### **Tcpdump**

for cli output

```shell-session
sudo tcpdump -i ens224 
```

pktmon.exe -> for windows 10 host. And remember to save the pcap traffic.

#### **Responder**

```bash
sudo responder -I ens224 -A 
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption><p>responder passive analysis mode.</p></figcaption></figure>

Responder might find more unique hosts from wireshark.

**Fping**

Fping is ping tool that sends ICMP packets multiple hosts at once and we can scprit it. It also works in round-robin fashion (cyclical manner -> not wainting for multiple requests from single host to return then move on)

```shell-session
fping -asgq 172.16.5.0/23
```

* a -> show targets that are alive
* s -> print stats at end of scan
* g -> generate target list form CIDR network
* q -> not show per-target results

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>3 hosts alive</p></figcaption></figure>

#### **Nmap**

To identify critical hosts (Domain Controllers and web servers), potential vulnerable host, etc. it is good idea to focus on AD specific protocols (DNS, SMB, LDAP, Kerberos, etc.)

```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

This will scan well-known ports, all of the hosts we found eirlier are added to hosts.txt file

```
# Nmap 7.92 scan initiated Sun Jul  2 23:43:31 2023 as: nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
Nmap scan report for inlanefreight.local (172.16.5.5)
Host is up (0.0017s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-07-03 03:44:07Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
|_ssl-date: 2023-07-03T03:46:01+00:00; -1s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
|_ssl-date: 2023-07-03T03:46:01+00:00; 0s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
|_ssl-date: 2023-07-03T03:46:01+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
|_ssl-date: 2023-07-03T03:46:01+00:00; 0s from scanner time.
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-02T03:13:44
| Not valid after:  2024-01-01T03:13:44
| MD5:   4fea da86 8ee3 12d6 250d c478 5103 971f
|_SHA-1: 54f4 306b c416 0673 0e66 e4af 5d25 654e 71dd 7364
|_ssl-date: 2023-07-03T03:46:01+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-DC01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2023-07-03T03:44:56+00:00
MAC Address: 00:50:56:B9:15:B8 (VMware)

```

From the scan result we foud NetBIOS and DNS naming standards along with few hosts has RDP open, which pointed us to the primary Domain Controller for INLANEFREIGHT.LOCAL domain -> ACADEMY-EA-**DC01**.INLANEFREIGHT.LOCAL

### Identifying Users

#### Kerbrute - Internal AD Username Enumeration

kerbrute takes advantage of Kerberos pre-authentication fails not triggering logs. We will use this alog with jsmith.txt or jsmith2.txt workdlist from [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames), which is a useful repo for enumerating users while we are unauthenticated.

**Compiling kerbrute**

```shell-session
sudo git clone https://github.com/ropnop/kerbrute.git # clone repo
make help # list all make options
sudo make all # complile all
ls dist/ # this dir will be created after compiling
```

**setting up kerbrute**

```shell-session
./kerbrute_linux_amd64 
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

```shell-session
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption><p>56 valid usernames</p></figcaption></figure>

### Identifying Potential Vulnerabilities

Having SYSTEM-level access within a domain environment is nearly equivalent to having a domain user account. To gain SYSTEM-level access on a host try following.

* Remote Windows exploits such as MS08-067, EternalBlue, or BlueKeep.
* Abusing a service running in the context of the `SYSTEM account`, or abusing the service account `SeImpersonate` privileges using [Juicy Potato](https://github.com/ohpe/juicy-potato). This type of attack is possible on older Windows OS' but not always possible with Windows Server 2019.
* Local privilege escalation flaws in Windows operating systems such as the Windows 10 Task Scheduler 0-day.
* Gaining admin access on a domain-joined host with a local account and using Psexec to launch a SYSTEM cmd window

After gaining SYSTEM-level access we could perform the following.

* Enumerate the domain using built-in tools or offensive tools such as **BloodHound** and **PowerView**.
* Perform **Kerberoasting** / **ASREPRoasting** attacks within the same domain.
* Run tools such as **Inveigh** to gather **Net-NTLMv2 hashes** or perform **SMB relay attacks**.
* Perform **token impersonation** to hijack a privileged domain user account.
* Carry out **ACL attacks**.

