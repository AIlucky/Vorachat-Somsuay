---
description: The vulnerablility caused by Insecure Coding
---

# Bypassing Security Filters

## Identification

The below example showcases the file name validation implementation in the back-end to prevent a command injection vulnerability.

If we try to inject a filename in the `New File Name` field with a special character we will be promted with a error shown below.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_malicious_request.jpg" alt=""><figcaption><p>filename 'test;'</p></figcaption></figure>

## Exploit

To exploite it we can try to chage the HTTP method from frist it was using POST to GET and see if the new file is created using a special character.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_GET_request.jpg" alt=""><figcaption><p>POST to GET</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_injected_request.jpg" alt=""><figcaption><p>SUCCESS</p></figcaption></figure>

To verify the exploit we can try to create multiple files using shell command and see if more than one file is created at once.

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_filter_bypass_request.jpg" alt=""><figcaption><p>file1; touch file2;</p></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_after_filter_bypass.jpg" alt=""><figcaption><p>SUCCESS</p></figcaption></figure>







