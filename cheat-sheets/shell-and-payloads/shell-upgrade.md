---
description: >-
  In some cases we will endup with a shell that is not interactive (usually
  known as jail shell). to upgrade it there are several methods.
---

# Shell upgrade

### /bin/sh -i

**Interactive**

```shell-session
/bin/sh -i
sh: no job control in this shell
sh-4.2$
```

### Perl

**Perl To Shell**

```shell-session
perl —e 'exec "/bin/sh";'
```

```sh
perl: exec "/bin/sh"; # needs to be ran from a script
```

### Python

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

### Ruby

**Ruby To Shell**

```shell-session
ruby: exec "/bin/sh" # needs to be ran from a script
```

### Lua

**Lua To Shell**

```shell-session
lua: os.execute('/bin/sh') # needs to be ran from a script
```

### AWK

**AWK To Shell**

```shell-session
awk 'BEGIN {system("/bin/sh")}'
```

### Find

**Using Find For A Shell**

```shell-session
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

### Find + Exec

```shell-session
find . -exec /bin/sh \; -quit
```

### VIM

**Vim To Shell**

```shell-session
vim -c ':!/bin/sh'
```

**Vim Escape**

```shell-session
vim
:set shell=/bin/sh
:shell
```

### To get a full interactive shell

1. `Ctrl-Z` backgrounding the shell process.
2. `stty -a` checking the number of rows and columns in the host terminal.
3. `stty raw -echo` setting terminal settings like new line, break characters etc.
4. `fg + ENTER` returning to the shell.
5. `reset`, `export SHELL=bash`, `export TERM=xterm-256color` declaring environment variables to be able to use cllear etc. and colors.
6. `stty rows <num> columns <cols>` setting the terminal rows and columns based on the host configuration.

