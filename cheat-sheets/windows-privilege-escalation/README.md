# 🪟 Windows Privilege Escalation

* Target -> get to `Local Administrator` group or `NT AUTHORITY\SYSTEM` accout
* Some of the ways to escalate the privilege
  * Abusing Windows group privileges
  * Abusing Windows user privileges
  * Bypassing User Account Control
  * Abusing weak service/file permissions
  * Leveraging unpatched kernel exploits
  * Credential theft
  * Traffic Capture

## Useful Tools

<table><thead><tr><th width="172">Tool</th><th>Description</th></tr></thead><tbody><tr><td><a href="https://github.com/GhostPack/Seatbelt">Seatbelt</a></td><td>C# project for performing a wide variety of local privilege escalation checks</td></tr><tr><td><a href="https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS">winPEAS</a></td><td>WinPEAS is a script that searches for possible paths to escalate privileges on Windows hosts. All of the checks are explained <a href="https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation">here</a></td></tr><tr><td><a href="https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1">PowerUp</a></td><td>PowerShell script for finding common Windows privilege escalation vectors that rely on misconfigurations. It can also be used to exploit some of the issues found</td></tr><tr><td><a href="https://github.com/GhostPack/SharpUp">SharpUp</a></td><td>C# version of PowerUp</td></tr><tr><td><a href="https://github.com/411Hall/JAWS">JAWS</a></td><td>PowerShell script for enumerating privilege escalation vectors written in PowerShell 2.0</td></tr><tr><td><a href="https://github.com/Arvanaghi/SessionGopher">SessionGopher</a></td><td>SessionGopher is a PowerShell tool that finds and decrypts saved session information for remote access tools. It extracts PuTTY, WinSCP, SuperPuTTY, FileZilla, and RDP saved session information</td></tr><tr><td><a href="https://github.com/rasta-mouse/Watson">Watson</a></td><td>Watson is a .NET tool designed to enumerate missing KBs and suggest exploits for Privilege Escalation vulnerabilities.</td></tr><tr><td><a href="https://github.com/AlessandroZ/LaZagne">LaZagne</a></td><td>Tool used for retrieving passwords stored on a local machine from web browsers, chat tools, databases, Git, email, memory dumps, PHP, sysadmin tools, wireless network configurations, internal Windows password storage mechanisms, and more</td></tr><tr><td><a href="https://github.com/bitsadmin/wesng">Windows Exploit Suggester - Next Generation</a></td><td>WES-NG is a tool based on the output of Windows' <code>systeminfo</code> utility which provides the list of vulnerabilities the OS is vulnerable to, including any exploits for these vulnerabilities. Every Windows OS between Windows XP and Windows 10, including their Windows Server counterparts, is supported</td></tr><tr><td><a href="https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite">Sysinternals Suite</a></td><td>We will use several tools from Sysinternals in our enumeration including <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk">AccessChk</a>, <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/pipelist">PipeList</a>, and <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/psservice">PsService</a></td></tr></tbody></table>

* pre-compiled binaries of `Seatbelt` and `SharpUp` [here](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)
* standalone binaries of `LaZagne` [here](https://github.com/AlessandroZ/LaZagne/releases/)

