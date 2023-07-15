---
description: open-source support ticketing system.
---

# osTicket

* osTicket can integrate user inquiries from email, phone, and web-based forms into a web interface
* osTicket is written in PHP and uses a MySQL backend
* It can be installed on Windows or Linux

## Footprinting/Discovery/Enumeration

![EyeWitness scan](https://academy.hackthebox.com/storage/modules/113/osticket\_eyewitness.png)

cookie named `OSTSESSID` was set when visiting the page.

* most osTicket installs will showcase the osTicket logo with the phrase `powered by` in front of it in the page's footer.
* The footer may also contain the words `Support Ticket System`.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/osticket_main.png" alt=""><figcaption><p>http://support.inlanefreight.local/</p></figcaption></figure>

Nmap scan will not help

`osTicket` is a web application that is highly maintained and serviced, we will not find many vulnerabilities and exploits

**break down the main functions**

| `1. User input` | `2. Processing` | `3. Solution` |
| --------------- | --------------- | ------------- |

**User Input**

* The core function of osTicket is to inform the company's employees about a problem
* the application is open-source
* many tutorials and examples available to take a closer look at the application
* only staff and users with administrator privileges can access the admin panel

**Processing**

* staff or administrators try to reproduce significant errors to find the core of the problem.
* Processing is finally done internally in an isolated environment very similar to production.
* if staff and admin suspect internal bug that may affect business, they will try to uncover more and address more issues.

**Solution**

* staff members from the technical departments will be involved in the email correspondence
* This will give us new email addresses to use against the osTicket admin panel
  * and potential usernames with which we can perform OSINT

## Attacking osTicket

* osTicket version 1.14.1 suffers from [CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881) which was an SSRF vulnerability
* we may leverage to gain access to internal resources or perform internal port scanning.

support portals can sometimes be used to obtain an email address for a company domain

Suppose we find an exposed service such as a company's Slack server or GitLab, which requires a valid company email address to join. Many companies have a support email such as `support@inlanefreight.local`, and emails sent to this are available in online support portals that may range from Zendesk to an internal custom tool.

If we come across a customer support portal we can try submitting new ticket and obtain valid company email.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/new_ticket.png" alt=""><figcaption><p>http://support.inlanefreight.local/open.php</p></figcaption></figure>

This is a modified version of osTicket as an example, but we can see that an email address was provided.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/ticket_email.png" alt=""><figcaption></figcaption></figure>

If the company set up their helpdesk software to correlate ticket numbers with emails, then any email sent to the email we received when registering, `940288@inlanefreight.local`, would show up here

<figure><img src="https://academy.hackthebox.com/storage/modules/113/ost_tickets.png" alt=""><figcaption><p>http://support.inlanefreight.local/open.php</p></figcaption></figure>

### osTicket - Sensitive Data Exposure

[Dehashed](http://dehashed.com/)

```shell-session
sudo python3 dehashed.py -q inlanefreight.local -p

id : 5996447501
email : julie.clayton@inlanefreight.local
username : jclayton
password : JulieC8765!
hashed_password : 
name : Julie Clayton
vin : 
address : 
phone : 
database_name : ModBSolutions


id : 7344467234
email : kevin@inlanefreight.local
username : kgrimes
password : Fish1ng_s3ason!
hashed_password : 
name : Kevin Grimes
vin : 
address : 
phone : 
database_name : MyFitnessPal

<SNIP>
```

This dump shows cleartext passwords for two different users: `jclayton` and `kgrimes`.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/osticket_admin.png" alt=""><figcaption><p>http://support.inlanefreight.local/scp/login.php</p></figcaption></figure>

we try `kevin@inlanefreight.local` and get a successful login!

<figure><img src="https://academy.hackthebox.com/storage/modules/113/osticket_kevin.png" alt=""><figcaption></figcaption></figure>

The user `kevin` appears to be a support agent but does not have any open tickets. we would expect to see some open tickets

<figure><img src="https://academy.hackthebox.com/storage/modules/113/osticket_ticket.png" alt=""><figcaption><p>conversation between a remote employee and the support agent.</p></figcaption></figure>

## Assessment

Find your way into the osTicket instance and submit the password sent from the Customer Support Agent to the customer Charles Smithson.

<figure><img src="../../../.gitbook/assets/ภาพ (3).png" alt=""><figcaption><p>674135@inlanefreight.local</p></figcaption></figure>

Login with `kevin@inlanefreight.local:Fish1ng_s3ason!`

<figure><img src="../../../.gitbook/assets/ภาพ (7).png" alt=""><figcaption><p>Inlane_welcome!</p></figcaption></figure>
