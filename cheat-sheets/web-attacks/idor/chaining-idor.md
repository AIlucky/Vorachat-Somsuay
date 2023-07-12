# Chaining IDOR

Basically keep updating the value of the modified request, There might be multiple requests per action. We have to keep updating the value for the change to take affect.

### Information Disclosure

![get\_another\_user](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_idor\_get\_another\_user.jpg)

### Modifying Other Users' Details

![modify\_another\_user](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_idor\_modify\_another\_user.jpg)

### Chaining Two IDOR Vulnerabilities

1. Enumerate all users to find admin privilege to then edit our role in Modifying other Users' Details setp.
2. After we know what the admin role is named we can modify the role of our user to the admin role

![modify\_our\_role](https://academy.hackthebox.com/storage/modules/134/web\_attacks\_idor\_modify\_our\_role.jpg)

Refreshing the page we will update the role of the user.
