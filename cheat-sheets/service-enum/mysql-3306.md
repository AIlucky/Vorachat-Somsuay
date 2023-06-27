# MySQL \[3306]

### Nmap

```
nmap -sV -Pn -vv --script=mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 <ip> -p 3306 ​ 
nmap -sV -Pn -vv --script=mysql* <ip> -p 3306 
```

### Local Access

if you gain access to target box and see mysql running , you can try to connect with it from target locally

```
mysql -u root 
# Connect to root without password

mysql -u root -p 
# A password will be asked

# Always test root:root credential
```

### Remote Access

```
mysql -h <Hostname> -u root

mysql -h <Hostname> -u root@localhost
```

### If running as root <a href="#if-running-as-root" id="if-running-as-root"></a>

If Mysql is running as root and you have acces, you can run commands:

```
mysql> select do_system('id');

mysql> \! sh
```

### Exploring database

<table><thead><tr><th width="267">Commands</th><th>Description</th></tr></thead><tbody><tr><td><code>show databases;</code></td><td>List all the databases.</td></tr><tr><td><code>use &#x3C;database>;</code></td><td>select the database from available databases.</td></tr><tr><td><code>show tables;</code></td><td>List all the tables.</td></tr><tr><td><code>describe &#x3C;table>;</code></td><td>To see what the structure of a table</td></tr><tr><td><code>SELECT &#x3C;column1>, &#x3C;column2> FROM &#x3C;table>;</code></td><td>print out all the vlaues from the table and columns specified.</td></tr><tr><td>SELECT * FROM &#x3C;table> WHERE &#x3C;column> = "&#x3C;str>";</td><td>list contents where search item is located.</td></tr></tbody></table>

