# HTTP Verb Tampering

## Intro

**Commonly used HTTP verbs other than Get and POST**

<table><thead><tr><th width="125">Verb</th><th>Description</th></tr></thead><tbody><tr><td><code>HEAD</code></td><td>Identical to a GET request, but its response only contains the <code>headers</code>, without the response body</td></tr><tr><td><code>PUT</code></td><td>Writes the request payload to the specified location</td></tr><tr><td><code>DELETE</code></td><td>Deletes the resource at the specified location</td></tr><tr><td><code>OPTIONS</code></td><td>Shows different options accepted by a web server, like accepted HTTP verbs</td></tr><tr><td><code>PATCH</code></td><td>Apply partial modifications to the resource at the specified location</td></tr></tbody></table>

## Cause

### Insecure Configurations

```xml
<Limit GET POST>
    Require valid-user
</Limit>
```

admin may use like above code which can still be bypassed using method like HEAD.

### Insecure Coding

```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

The above code is the sanitization of SQL injection which seems to be only checking GET request.

* If GET request contains no bad characters execute the $query
* The $query uses $\_REQUEST\["code"] which may contain POST paramters
  * `leading to an inconsistency in the use of HTTP Verbs`

Attacker may use POST request to perform SQL injection.

* GET request will be empty (no bad charracters) and execute the code from post request.



