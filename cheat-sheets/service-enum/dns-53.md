# DNS \[53]

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

## Enumeration

```shell-session
nmap -p53 -Pn -sV -sC 10.10.110.213

--script dns-nsid
# To grab a banner
```

### Subdomain Enumeration

```bash
./subfinder -d inlanefreight.com -v

echo "ns1.inlanefreight.com" > ./resolvers.txt
./subbrute inlanefreight.com -s ./names.txt -r ./resolvers.txt

dnsenum --dnsserver <IP_DNS> --enum -p 0 -s 0 -o subdomains.txt -f subdomains-1000.txt <DOMAIN>
dnsrecon -D subdomains-1000.txt -d <DOMAIN> -n <IP_DNS>
dnscan -d <domain> -r -w subdomains-1000.txt #Bruteforce subdomains in recursive way, https://github.com/rbsec/dnscan

host -l -a <example.com> <target_ip>
```

## DNS Zone Transfer

```bash
dig axfr @<DNS_IP>

dig axfr @<DNS_IP> <DOMAIN>
dig AXFR @ns1.inlanefreight.htb inlanefreight.htb

fierce --domain zonetransfer.me
fierce --domain <DOMAIN> --dns-servers <DNS_IP>
```

## More Info

```bash
dig ANY @<DNS_IP> <DOMAIN>     #Any information
dig A @<DNS_IP> <DOMAIN>       #Regular DNS request
dig AAAA @<DNS_IP> <DOMAIN>    #IPv6 DNS request
dig TXT @<DNS_IP> <DOMAIN>     #Information
dig MX @<DNS_IP> <DOMAIN>      #Emails related
dig NS @<DNS_IP> <DOMAIN>      #DNS that resolves that name
dig -x 192.168.0.2 @<DNS_IP>   #Reverse lookup
dig -x 2a00:1450:400c:c06::93 @<DNS_IP> #reverse IPv6 lookup

#Use [-p PORT]  or  -6 (to use ivp6 address of dns)
```

