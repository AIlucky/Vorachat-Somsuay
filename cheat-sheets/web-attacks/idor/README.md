---
description: Insecure Direct Object References (IDOR)
---

# IDOR

## Intro

The most basic example of an IDOR vulnerability is accessing private files and resources of other users that should not be accessible to us, like personal files or credit card data, which is known as `IDOR Information Disclosure Vulnerabilities`.

IDOR vulnerabilities may also lead to the elevation of user privileges with `IDOR Insecure Function Calls`.

For example, many web applications expose URL parameters or APIs for admin-only functions in the front-end code of the web application and disable these functions for non-admin users. However, if we had access to such parameters or APIs, we may call them with our standard user privileges. Suppose the back-end did not explicitly deny non-admin users from calling these functions. In that case, we may be able to perform unauthorized administrative operations, like changing users' passwords or granting users certain roles, which may eventually lead to a total takeover of the entire web application.

## Identification

### URL Parameters & APIs

e.g. `?uid=1` or `?filename=file_1.pdf`. These are mostly found in URL parameters or APIs but may also be found in other HTTP headers, like cookies.

We can try incrementing the values to retrive other data e.g. `?uid=2`or`?filename=file_2.pdf`

We can also use a fuzzing application to try thousands of variations and see if they return any data. **Any successful hits to files that are not our own would indicate an IDOR vulnerability.**

### AJAX Calls

Identify unused parameters or APIs in front-end code from JavaScript AJAX calls. Some JS framework may disclose function calls on front-end and use them based on roles.

e.g. while we are low priv user, we can still look at the admin function and identify AJAX calls to specifi end-points or APIs.

### Hashing and Encoding

Some web app may encode the reference or hash it the sequential values. If we identify them, we may be able to exploit them. (if back-end has no access control system)

`?filename=ZmlsZV8xMjMucGRm` -> we can guess that file name is base64, we can encode new value and pass in to test IDOR

### User Roles

Test as many user roles as possible, and compare the HTTP requests and object reerences. This may give us different endpoints and API calls for us to try with low privilege user.
