# Attacking common services

## Assessment

### FTP

What username is available for the FTP server?

* I used the wordlist from ftp **anonymous** and get the username and password list.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

### SMB

What is the name of the shared folder with READ permissions?

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption><p>GGJ</p></figcaption></figure>

What is the password for the username "jason"?

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>MAKE SURE TO RUN WITH --local-auth</p></figcaption></figure>

Login as the user "jason" via SSH and find the flag.txt file. Submit the contents as your answer.

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

### SQL DB

What is the password for the "mssqlsvc" user?

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

`mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:5a04b004fa4e9d85f6186628bbb5f628:0101000000000000800f5b3cd1a9d9013832986d3a5b90e8000000000100100045006c0046004800450079006a0061000300100045006c0046004800450079006a006100020010006d007100780051006100480062004900040010006d00710078005100610048006200490007000800800f5b3cd1a9d90106000400020000000800300030000000000000000000000000300000205c323361b6fee9491d3dade42cd1a0ade509215af472deb6f497f224759ddc0a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310034002e0034000000000000000000`

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Enumerate the "flagDB" database and submit a flag as your answer.

```
impacket-mssqlclient 'mssqlsvc:princess1@10.129.203.12' -windows-auth
```

* Then explored the flagDB and got the hash.
