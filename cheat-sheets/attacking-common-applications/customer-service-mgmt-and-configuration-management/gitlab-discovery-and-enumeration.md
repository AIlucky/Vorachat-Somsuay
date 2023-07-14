---
description: >-
  web-based Git-repository hosting tool that provides wiki capabilities, issue
  tracking, and continuous integration and deployment pipeline functionality
---

# Gitlab - Discovery & Enumeration

* open-source and written in Go, Ruby on Rails, and Vue.js.
* GitLab is similar to GitHub and BitBucket, which are also web-based Git repository tools
* These Git repositories may hold publicly available code such as scripts to interact with an API
* we may also find scripts or configuration files that were accidentally committed containing cleartext secrets such as passwords that we may use to our advantage

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_signup_res.png" alt=""><figcaption><p>http://gitlab.inlanefreight.local:8081/admin/application_settings/general</p></figcaption></figure>

Two-factor authentication is disabled by default.

## Footprinting & Discovery

We can quickly determine that GitLab is in use in an environment by just browsing to the GitLab URL, and we will be directed to the login page, which displays the GitLab logo.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_login.png" alt=""><figcaption><p>http://gitlab.inlanefreight.local:8081/users/sign_in</p></figcaption></figure>

* The only way to footprint the GitLab version number in use is by browsing to the `/help` page when logged in
* If the GitLab instance allows us to register an account, we can log in and browse to this page to confirm the version.
* If we cannot register an account, we may have to try a low-risk exploit such as [this](https://www.exploit-db.com/exploits/49821).
* if we have no way to enumerate the version number, try hunting for secrets and not try multiple exploits against it blindly
* There have been a few serious exploits against GitLab [12.9.0](https://www.exploit-db.com/exploits/48431) and GitLab [11.4.7](https://www.exploit-db.com/exploits/49257) in the past few years as well as GitLab Community Edition [13.10.3](https://www.exploit-db.com/exploits/49821), [13.9.3](https://www.exploit-db.com/exploits/49944), and [13.10.2](https://www.exploit-db.com/exploits/49951).

## Enumeration

If can't enum version number, try browsing to `/explore` and see if there are any public projects that may contain something interesting.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_explore.png" alt=""><figcaption><p>project Inlanefreight dev</p></figcaption></figure>

Browsing to the project, it looks like an example project and may not contain anything useful, though it is always worth digging around.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_example.png" alt=""><figcaption><p>http://gitlab.inlanefreight.local:8081/root/inlanefreight-dev</p></figcaption></figure>

* explore each of the pages linked in the top right `groups`, `snippets`, and `help`
* use the search functionality and see if we can uncover any other projects.
* check and see if we can register an account and access additional projects

organization did not set up GitLab only to allow company emails to register or require an admin to approve a new account. In that case, we may be able to access additional data.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_signup.png" alt=""><figcaption><p>http://gitlab.inlanefreight.local:8081/users/sign_up</p></figcaption></figure>

* Use the enumeration form to enum valid user
  *   Then guess weak password from the list of users

      <figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_taken2.png" alt=""><figcaption><p>user <code>root</code> is taken</p></figcaption></figure>
* If register with taken email -> `1 error prohibited this user from being saved: Email has already been taken`
* User enum works with latest gitlab version

Some mitigations can be put in place for this, such as enforcing 2FA on all user accounts, using `Fail2Ban` to block failed login attempts which are indicative of brute-forcing attacks, and even restricting which IP addresses can access a GitLab instance if it must be accessible outside of the internal corporate network.

Let's go ahead and register with the credentials `hacker:Welcome` and log in and poke around.

<figure><img src="https://academy.hackthebox.com/storage/modules/113/gitlab_internal.png" alt=""><figcaption><p>Internal projecct Inlane freight website</p></figcaption></figure>

we could possibly download the source and review it for vulnerabilities or hidden functionality or find credentials or other sensitive data.

In a real-world scenario, we may be able to find a considerable amount of sensitive data if we can register and gain access to any of their repositories. As this [blog post](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html) explains, there is a considerable amount of data that we may be able to uncover on GitLab, GitHub, etc.

## Assessment

Enumerate the GitLab instance at http://gitlab.inlanefreight.local. What is the version number?

<figure><img src="../../../.gitbook/assets/ภาพ (3).png" alt=""><figcaption><p>create gitlab acount and go to help page.</p></figcaption></figure>

Find the PostgreSQL database password in the example project.

<figure><img src="../../../.gitbook/assets/ภาพ (5).png" alt=""><figcaption><p>postgres</p></figcaption></figure>





