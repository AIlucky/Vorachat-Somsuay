# Skills Assessment

**Try to escalate your privileges and exploit different vulnerabilities to read the flag at '/flag.php'.**

Profile button sends the following request

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>Profile</p></figcaption></figure>

We can enumerate uid 1-100 and find the admin user and look for the keyword admin.

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption><p>only one user with Administrator</p></figcaption></figure>

Try changing password with for the Administrator account

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Then for the request we change from POST to GET

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption><p>password changed successfully</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption><p>we are now logged in as admin and ADD EVENT function is not there before</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>create event request seems to be injection point.</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>at the name tag</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption><p>got local file inclusion using XXE</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>HTB{m4573r_w3b_4774ck3r}</p></figcaption></figure>
