# Intro to SQL Injections

## Structured Query Language (SQL)

* SQL syntax can differ from RDBMS
* They are all required to follow the [ISO standard](https://en.wikipedia.org/wiki/ISO/IEC\_9075)

### SQL Injection

An SQL injection occurs when user-input is inputted into the SQL query string without properly sanitizing or filtering the input

```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

In typical cases, the `searchInput` would be inputted to complete the query, returning the expected outcome. Any input we type goes into the following SQL query:

```sql
select * from logins where username like '%$searchInput'
```

**we can add a single quote (`'`), which will end the user-input field, and after it, we can write actual SQL code**. For example, if we search for `1'; DROP TABLE users;`, the search input would be:

```php
'%1'; DROP TABLE users;'
```

```sql
select * from logins where username like '%1'; DROP TABLE users;'
```

<img src="../../.gitbook/assets/file.excalidraw (2).svg" alt="Types of SQL Injections" class="gitbook-drawing">

* In-band SQL injection -> output of both intended and new query printed on frontend.
  * Union Based -> have to specify the exact location 'i.e., column'
  * Error Based -> used when we get the PHP or SQL errors in fromt-end, can intentionally cause error that returns output of query
* Blind SQL injection -> output is not printed, retrive output character by character
  * Boolean Based -> control whether page returns any output at all to check true false
  * Time Based -> conditional statements delay the page response if true
* Out-of-band SQL injection -> no output at all, have to direct output to remote location.
