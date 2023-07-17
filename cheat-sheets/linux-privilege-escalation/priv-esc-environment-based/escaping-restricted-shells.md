---
description: >-
  A restricted shell is a type of shell that limits the user's ability to
  execute commands.
---

# Escaping Restricted Shells

* user can execute specific sets of command / in a specific directory
* Restricted Shells used to provide safe environment for users who might damage system or only allow a user to access certain system features

## Restricted shells

### **RBASH**

[Restricted Bourne shell](https://www.gnu.org/software/bash/manual/html\_node/The-Restricted-Shell.html) (`rbash`) limits the users' ability to use a certain features of BASH, often used to provide safe and contrlled environment for users who may damage the system.

### **RKSH**

[Restricted Korn shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) (`rksh`) A restricted version of Korn shell, limist the aility to use certain features of Korn shell.

### **RZSH**

[Restricted Z shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) (`rzsh`) is a restricted version of the Z shell and is the most powerful and flexible command-line interpreter.

Administrators often use restricted shells in enterprise networks. By limiting the user's ability to execute specific commands or access certain directories, administrators can ensure that users cannot perform actions that could harm the system or compromise the network's security.

## Escaping

It is possible in some cases to escape the restricted shell by injecting commands to command line or other imputs.

* If shell allows user to execute commands by passing them as arugments to built-in commands then we can try injection addition commands in to the argument.

**Command injection**

Suppose restricted shell allow us to pass in arguments for ls command e.g. -l -a. In this situation we can perform command injection to escape from shell

```shell-session
ls -l `pwd` 
```

* `ls -l` will be executed
* &#x20;Then `pwd` will be executed since `pwd` is not restricted.

**Command Substitution**

Using the shell's command substitution syntax to execute a command

E.G. shell allows users to execute commands by enclosing them in backticks `` ` `` . It may be possible to escape from the shell by executing a command in a backtick substitution that is not restricted by the shell.

**Command Chaining**

We would need to use multiple commands in a single command line, separated by a shell metacharacter, such as a semicolon (`;`) or a vertical bar (`|`), to execute a command.

**Environment Variables**

modifying or creating environment variables that the shell uses to execute commands that are not restricted by the shell.

* if the shell uses an environment variable to specify the directory in which commands are executed, it may be possible to escape from the shell by modifying the value of the environment variable to specify a different directory.

**Shell Functions**

define and call shell functions that execute commands not restricted by the shell. Let us say, the shell allows users to define and call shell functions, it may be possible to escape from the shell by defining a shell function that executes a command.

## Assessment

* Find commands that are available to us
  * echo | pwd&#x20;
* Since echo is available, we can use echo \* to list the contents of the directory.

```
htb-user@ubuntu:~$ echo *
bin flag.txt
```

* Once the flag was found we can use echo to read the contents of the file. Since echo can't read the STDIN it's meaningless to pipe the command, we can try the following

```
tb-user@ubuntu:~$ echo "$(<flag.txt)"
HTB{35c4p3_7h3_r3stricted_5h311}
```

```
ssh psj@server_name-t "bash --noprofile"
```

This also works
