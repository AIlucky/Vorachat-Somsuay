---
description: >-
  a type of security vulnerability where attackers can modify the model
  attributes of an application through the parameters sent to the server
---

# Web Mass Assignment Vulnerabilities

Ruby on Rails is a web application framework that is vulnerable to this type of attack.

Assuming we have a `User` model with the following attributes:

Code: ruby

```ruby
class User < ActiveRecord::Base
  attr_accessible :username, :email
end
```

* above code, only the `username` and `email` attributes are allowed to be mass-assigned.&#x20;
* attackers can modify other attributes by tampering with the parameters sent to the server.&#x20;

Let's assume that the server receives the following parameters.

```javascript
{ "user" => { "username" => "hacker", "email" => "hacker@example.com", "admin" => true } }
```

Although the `User` model does not explicitly state that the `admin` attribute is accessible, the attacker can still change it because it is present in the arguments. Bypassing any access controls that may be in place, the attacker can send this data as part of a POST request to the server to establish a user with admin privileges.

### Exploiting Mass Assignment Vulnerability

![pending](https://academy.hackthebox.com/storage/modules/113/mass\_assignment/pending.png)

Consider the above example, we login by sending username and password value to the server but the server responce was pending for approval, this means that the account needs validation

If we get a hand on the source code of backend or able to guess the parameter used by other privilege then we could try to send the same request along with the hidden thus the request would be

```
username=new&password=test&confirmed=test
```

Like so, we could successfully login and completly bypass the validation process.

## Assessment

**We placed the source code of the application we just covered at /opt/asset-manager/app.py inside this exercise's target, but we changed the crucial parameter's name. SSH into the target, view the source code and enter the parameter name that needs to be manipulated to log in to the Asset Manager web application.**

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

