# Intro to AD enum & attacks

## AD background

* Ad is a directory server for Windows enterprise.
* AD was implemented in 2000
* AD is based on x.500 and LDAP
* AD is used for centralized management which includes
  * users
  * computers
  * groups
  * network devices (file shares)
  * policies
  * devices
* AD provides **authentication**, **accounting**, and **authorization**

## Why Care?

* AD holds very high market share fo **Identity and access management** solution
* In last 2 Years it has 2k reported CVE
* Because it is easy for end users, a simple misconfig expsoes the company to vulnerabilities

{% hint style="info" %}
The general goal of gaining a foothold in a client's AD environment is to `escalate privileges` by moving laterally or vertically throughout the network
{% endhint %}

## Real world examples

{% tabs %}
{% tab title="Scenario 1 - Waiting On An Admin" %}
After compromising a host and gaining SYSTEM level access, we can enumerate the domain right away if the system is domain-joined. If we found the Service Principal Names (SPNs) in the env, we can perform **Kerberoasting** attack to retrive TGS. Using hashcat we cat try to crack the TGS. We might get a hit and pivot to another host that in this scenario has the write access to SMB share and we could perfrom SCF file attack. If we are lucky we could get a hit with the users' hash and using **BloodHound** we can see the privilege of the hit user.
{% endtab %}

{% tab title="Scenario 2 - Spraying" %}
Password spraying could lead us to foothold in a domain but beware of account lock outs. In this scenario the writer found SMB null session usign **enum4linux**. He gained a listing of all users and domain password policy. By having access to the password policy, he could know the, minimum lenght, how many tries before lock outs, what characters. He used that info and guessed **Spring@18** and got a hit. Running Blood hound let him know what access the user has access to as local admin. He noticed a domain admin account had active session on those hosts. He then used Rubeus to extract TGT from domain admin user. Then he used PTH attack and authenticate as admin.
{% endtab %}

{% tab title="Scenario 3 - Fighting In The Dark " %}
In this scenario the writer used **linkedin2username** tool to generate user name from company's linkedin page. He used those username generated along with **statistically-likely-usernames** repo. After enum with **Kerbrute**, he found 516 valid users. He then performed password spraying with Welcome2021 password and got a hit. He use the account to run python version of BloodHound and found all domain users has RDP access to one box. He logged in to the host and used **DomianPasswrdSpray** to spray with password policy (DomainPasswordSpray tool removes the near lock out users from the list). He was authenticated and spray all domain users, which gave him more hits. He checked the priv of all accounts and found that one was in the Help DEsk group which has **GenericAll** rights over the **Enterprise Key Admins** group, and **Enterprise Key Admins** group had **GenericAll** priv over **domain controller**. He added the account to this group, authenticated again, and inherited the privileges. Using those privs, he performed **Shadow Credentials** attack and got NT hash for domian controller account. Using the hash he perform a DCSync attack and got NTLM password hashes for all users in the domain (a domain controller can perform replication)
{% endtab %}
{% endtabs %}

