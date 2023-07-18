# Situational Awareness

### Network Information

We should always look at [routing tables](https://en.wikipedia.org/wiki/Routing\_table) to view information about the local network and networks around it. If the host is part of AD we could gather info about local domain and IP from DC.

View ARP cache for each interface to see which hosts recently communicated with us.

**Interface(s), IP Address(es), DNS Information**

```cmd-session
C:\htb> ipconfig /all
```

**ARP Table**

```cmd-session
C:\htb> arp -a
```

**Routing Table**

```cmd-session
C:\htb> route print
```

### Enumerating Protections

**Check Windows Defender Status**

```powershell-session
PS C:\htb> Get-MpComputerStatus
```

**List AppLocker Rules**

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

**Test AppLocker Policy**

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone

FilePath                    PolicyDecision MatchingRule
--------                    -------------- ------------
C:\Windows\System32\cmd.exe         Denied c:\windows\system32\cmd.exe
```

