---
description: Plink, short for PuTTY Link (If we had to use Windows attack host)
---

# SSH for Windows: plink.exe

Windows did not have a native ssh client included, so users would have to install their own. The tool of choice for many a sysadmin who needed to connect to other hosts was [PuTTY](https://www.putty.org/).

**Using Plink.exe**

```cmd-session
plink -D 9050 ubuntu@10.129.15.50
```

[Proxifier](https://www.proxifier.com) can be used to start a SOCKS tunnel via the SSH session we created.

![Proxifier](https://academy.hackthebox.com/storage/modules/158/reverse\_shell\_9.png)
