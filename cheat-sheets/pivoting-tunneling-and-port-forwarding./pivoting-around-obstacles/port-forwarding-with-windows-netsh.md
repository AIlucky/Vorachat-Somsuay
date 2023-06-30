---
description: >-
  Windows command-line tool that can help with the network configuration of a
  particular Windows system
---

# Port Forwarding with Windows Netsh

**Using Netsh.exe to Port Forward**

<pre class="language-cmd-session"><code class="lang-cmd-session"># netsh.exe interface portproxy add v4tov4 listenport=&#x3C;pivot port> listenaddress=&#x3C;pivot IP> connectport=&#x3C;target port> connectaddress=&#x3C;target IP>
<strong>C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.25
</strong></code></pre>

**Verifying Port Forward**

```cmd-session
C:\Windows\system32> netsh.exe interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.42.198   8080        172.16.5.25     3389
```

**Connecting with xfreerdp**

```
xfreerdp /v:10.129.88.112:8080 /u:victor /p:'pass@123'
```

