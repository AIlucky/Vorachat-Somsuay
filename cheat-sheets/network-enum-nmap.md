# 🌐 Network Enum (Nmap)

### General port/service TCP scan

<pre class="language-bash" data-full-width="true"><code class="lang-bash"><strong>sudo nmap &#x3C;ip> -p- -sC -sV -oA &#x3C;output>
</strong></code></pre>

### Rust scan

```bash
rustscan -a <ip>
```

{% hint style="info" %}
rustscan is usually faster than nmap but the scans might not be accurate or complete. I prefer to use rustscan to scan all ports and then enumerate the service accordingly in nmap.
{% endhint %}

```bash
sudo nmap <ip> -p <ports from rustscan> -sC -sV -oA <output>
```

### UDP scan

```bash
sudo nmap <ip> -sU -O -p- -oA <output>
```

As far as I know, there are only a handful of tools that could perform UDP scans. UDP scans are slower than TCP scans in general.

### Scanning top ports

```bash
sudo nmap <ip> --top-ports=<num> -sC -sV -oA <output>
# selective of the most popular ports where num is the number of ports
```

```bash
sudo nmap <ip> -F -sC -sV -oA <output>
# -F is top 100
```

### Scans to avoide Firewalls, IDS, and IPS

<pre class="language-sh"><code class="lang-sh"><strong>sudo nmap &#x3C;ip> -n -Pn -p &#x3C;port> -O -S &#x3C;src ip> -e &#x3C;interface>
</strong><strong># sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
</strong></code></pre>

```bash
sudo nmap <ip> -p <port> -sS -Pn -n --disable-arp-ping --source-port <src port>
# sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --source-port 53
```

```bash
sudo nmap <ip> -p <port> -sS -Pn -n --disable-arp-ping -D RND:<#ips>
# sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping -D RND:5
```
