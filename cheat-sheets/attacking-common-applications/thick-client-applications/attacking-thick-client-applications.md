---
description: >-
  Thick client applications are the applications that are installed locally on
  our computers.
---

# Attacking Thick Client Applications

* usually applications used in enterprise environments created to serve specific purposes.
* These applications are usually developed using Java, C++, .NET, or Microsoft Silverlight.
* Examples of thick client applications are web browsers, media players, chatting software, and video games.

![arch\_tiers](https://academy.hackthebox.com/storage/modules/113/thick\_clients/arch\_tiers.png)

### Penetration Testing Steps

**Information Gathering**

* identify the application architecture, the programming languages and frameworks that have been used, and understand how the application and the infrastructure work.
* The following tools will help us gather information.
  * [CFF Explorer](https://ntcore.com/?page\_id=388)
  * [Detect It Easy](https://github.com/horsicq/Detect-It-Easy)
  * [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)
  * [Strings](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)

**Client Side attacks**

Interaction with servers and other external systems can expose thick clients to vulnerabilities similar to those found in web applications, including command injection, weak access control, and SQL injection.

Tools

* <table data-header-hidden><thead><tr><th width="148"></th><th width="181"></th><th width="164"></th><th></th></tr></thead><tbody><tr><td><a href="https://www.ghidra-sre.org/">Ghidra</a></td><td><a href="https://hex-rays.com/ida-pro/">IDA</a></td><td><a href="http://www.ollydbg.de/">OllyDbg</a></td><td><a href="https://www.radare.org/r/index.html">Radare2</a></td></tr><tr><td><a href="https://github.com/dnSpy/dnSpy">dnSpy</a></td><td><a href="https://x64dbg.com/">x64dbg</a></td><td><a href="https://github.com/skylot/jadx">JADX</a></td><td><a href="https://frida.re/">Frida</a></td></tr></tbody></table>

**Network Side Attacks**

capture sensitive information that might be transferred through HTTP/HTTPS or TCP/UDP connection, and give us a better understanding of how that application is working

* Wireshark&#x20;
* tcpdump&#x20;
* TCPView&#x20;
* Burp Suite

**Server Side Attacks**

similar to web application attacks, pay attention to the most common ones including most of the OWASP Top Ten.
