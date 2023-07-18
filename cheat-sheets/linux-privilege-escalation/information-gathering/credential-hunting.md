# Credential Hunting

The /var directory typically contains the web root for whatever web server is running on the host. e.g. wordpress configuration containing mysql database credentials.

```shell-session
cat wp-config.php | grep 'DB_USER\|DB_PASSWORD'
```

The spool or mail directories, if accessible, may also contain valuable information or even credentials.

## SSH Keys

We may locate a private key for another, more privileged, user that we can use to connect back to the box with additional privileges.

Whenever finding SSH keys check the `known_hosts` file to find targets.

* This file contains a list of public keys for all the hosts which the user has connected to in the past and may be useful for lateral movement or to find data on a remote host that can be used to perform privilege escalation on our target.

## Assessment

Find the WordPress database password.

<figure><img src="../../../.gitbook/assets/image (15) (4).png" alt=""><figcaption><p>W0rdpr3ss_sekur1ty!</p></figcaption></figure>

