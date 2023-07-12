---
description: IDOR vulnerabilities may also exist in function calls and APIs
---

# IDOR in Insecure APIs

`IDOR Insecure Function Calls` enable us to call APIs or execute functions as another user. Such functions and APIs can be used to change another user's private information, reset another user's password, or even buy items using another user's payment information

### Identifying Insecure APIs

Using the same web as previous IDOR attacks, access Edit Profile

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_employee_manager.jpg" alt=""><figcaption><p>Employee Manager</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_edit_profile.jpg" alt=""><figcaption><p>Full name, Email, About Me</p></figcaption></figure>

Clicking Update profile and check the proxy app.

![update\_request](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_idor\_update\_request.jpg)

We see that it is using `PUT` to `/profile/api.php/profile/1` ,&#x20;

* PUT is used to update item
* POST is used to create new items
* DELETE is used to delete item
* GET is used to retrive item details

So, PUT method here is normal.

```json
{
    "uid": 1,
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "employee",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}
```

The interesting part of this request is in the body and the cookie, we can see that in the body there are uid, uuid, and role which was not present in the front-end form. Also, the cookie value also indicates the role of the user which we could manipulate to desired value if known the high privilge role.

### Exploiting

We can try to

1. Change uid to another users' and take over the account
2. Change users' details may allow us to perfomr several web attacks
3. Create new users with arbitrary details, or delete existing users
4. Change our role to a more privileged role (e.g. `admin`) to be able to perform more action

<figure><img src="../../../.gitbook/assets/image (96) (1).png" alt=""><figcaption><p>Assessment</p></figcaption></figure>































