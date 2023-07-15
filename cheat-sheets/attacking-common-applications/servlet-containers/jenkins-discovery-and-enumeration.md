---
description: >-
  open-source automation server written in Java that helps developers build and
  test their software projects continuously.
---

# Jenkins - Discovery & Enumeration

## Intro

* a server-based system that runs in servlet containers such as Tomcat
* Jenkins is a [continuous integration](https://en.wikipedia.org/wiki/Continuous\_integration) server

## Discovery/Footprinting

* it is often installed on Windows servers running as the all-powerful SYSTEM account.
* If we can gain access via Jenkins and gain remote code execution as the SYSTEM account.
* Jenkins runs on Tomcat port 8080 by default.
* It utilizes port 5000 to attach slave servers.
  * This port is used to communicate between masters and slaves.
* Jenkins can use a local database, LDAP, Unix user database, delegate security to a servlet container, or use no authentication at all.
* Administrators can also allow or disallow users from creating accounts.

## Enumeration

<figure><img src="https://academy.hackthebox.com/storage/modules/113/jenkins_global_security.png" alt=""><figcaption></figcaption></figure>

The default installation typically uses Jenkins’ database to store credentials and does not allow users to register an account. We can fingerprint Jenkins quickly by the telltale login page.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/jenkins_login.png" alt=""><figcaption></figcaption></figure>

We may encounter a Jenkins instance that uses weak or default credentials such as `admin:admin` or does not have any type of authentication enabled.

## Assessment

Log in to the Jenkins instance at http://jenkins.inlanefreight.local:8000. Browse around and submit the version number when you are ready to move on.

<figure><img src="../../../.gitbook/assets/ภาพ (6).png" alt=""><figcaption><p>2.303.1</p></figcaption></figure>
