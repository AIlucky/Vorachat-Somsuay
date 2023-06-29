# SMTP \[25]

## Enumeration

```
nmap 192.168.1.101 --script=smtp* -p 25

nmap --script=smtp-commands,smtp-enum-users,smtp-vuln-cve2010-4344,smtp-vuln-cve2011-1720,smtp-vuln-cve2011-1764 -p 25 $ip
```

### User Enumeration

```
smtp-user-enum -M VRFY -U unix_users.txt -t <ip>
smtp-user-enum -M VRFY -U footprinting-wordlist.txt -t 10.129.234.112 -v -w 30
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

for server in $(cat smtpmachines); do echo "******************" $server "*****************"; smtp-user-enum -M VRFY -U userlist.txt -t $server;done #for multiple servers
# For multiple servers
```

## Connection

```
 telnet <ip> 25
```

### Check if a user exists

```
VRFY root
```

### Ask the server if a user belongs to a mailing list

<pre><code><strong>EXPN root
</strong></code></pre>

### Identifies the recipient of the email message

```
RCPT TO:julio

550 5.1.1 julio... User unknown

RCPT TO:john

250 2.1.5 john... Recipient ok
```

### Brute Force <a href="#brute-force" id="brute-force"></a>

```
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -V
hydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL

hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```
