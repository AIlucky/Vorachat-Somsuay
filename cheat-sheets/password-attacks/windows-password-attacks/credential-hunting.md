# Credential Hunting

**Running Lazagne All**

```cmd-session
C:\Users\bob\Desktop> start lazagne.exe all
```

**Using findstr**

```cmd-session
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

**Places we should keep in mind when credential hunting:**

* Passwords in Group Policy in the SYSVOL share
* Passwords in scripts in the SYSVOL share
* Password in scripts on IT shares
* Passwords in web.config files on dev machines and IT shares
* unattend.xml
* Passwords in the AD user or computer description fields
* KeePass databases --> pull hash, crack and get loads of access.
* Found on user systems and shares
* Files such as pass.txt, passwords.docx, passwords.xlsx found on user systems, shares, [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)











