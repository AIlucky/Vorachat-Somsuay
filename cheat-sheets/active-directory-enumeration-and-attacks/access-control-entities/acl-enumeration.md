---
description: enumerating ACLs using PowerView and BloodHound.
---

# ACL Enumeration

## Enumerating ACLs with PowerView

**Using Find-InterestingDomainAcl**

```powershell-session
PS C:\htb> Find-InterestingDomainAcl
```

{% hint style="danger" %}
This will give massive amount of information.
{% endhint %}

To make use of the function and get what we need fire we need we have to know what we want.

For example, let's find if the user has any interesting ACL rights that we could take advantage of. First we need to get the SID of the target user.

```powershell-session
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley
```

### Powerview

Then we can use `Get-DomainObjectACL` function to perform targeted search.

**Using Get-DomainObjectACL**

```powershell-session
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None
```

if we search without the flag `ResolveGUIDs`, we will see results like the above, where the right `ExtendedRight` does not give us a clear picture of what ACE entry the user `wley` has because the ObjectAceType is returning GUID value, which is not human readable.

**Performing a Reverse Search & Mapping to a GUID Value**

```powershell-session
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

Name              : User-Force-Change-Password
DisplayName       : Reset Password
DistinguishedName : CN=User-Force-Change-Password,CN=Extended-Rights,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529
```

or we can just google the GUID and find what rights they have. But the better approach is to use `ResolveGUIDs` flag.

**Using the -ResolveGUIDs Flag**

```powershell-session
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 

AceQualifier           : AccessAllowed
ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : User-Force-Change-Password
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0
```

As presented in the output, the GUID was converted to human readable text, we could now know that the user wley has the right User-Force-Change-Password

### **Another manual method**

**Creating a List of Domain Users**

```powershell-session
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

{% hint style="warning" %}
The command can take a long time to run, powerview was alot faster.
{% endhint %}

**A Useful foreach Loop**

```powershell-session
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}

Path                  : Microsoft.ActiveDirectory.Management.dll\ActiveDirectory:://RootDSE/CN=Dana 
                        Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
InheritanceType       : All
ObjectType            : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : ObjectAceTypePresent
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\wley
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
```

Once we have this data, we could follow the same methods shown above to convert the GUID to a human-readable format to understand what rights we have over the target user.

### **Further Enumeration of Rights Using damundsen**

```powershell-session
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
AccessMask            : 131132
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

We can see `damundsen` has `GenericWrite` privileges over `Help Desk Level 1 group`. This means we can add any user to this group and inherit any rights that this group has.

Let's see if this group is nested into any other groups.

**Investigating the Help Desk Level 1 Group with Get-DomainGroup**

```powershell-session
PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

memberof                                                                      
--------                                                                      
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
```

**Recap**

* We have control over wley account
* We enum over objects that wley has control over and found force change password right over damundsen
* we found damundsen can add a member to Help Desk Level 1
* Help Desk Level 1 is nested to Information Technology gorup. Help Desk Level 1 will inherit any rights given to the Information Technology group.

Now let's check if members of Information Technology can do anything iteresting.

**Investigating the Information Technology Group**

```powershell-session
PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology"
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-1164
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-4016
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed

```

We can see that the user adunn has the GenericAll rights which means we could.

* Modify group membership
* Force change a password
* Perform a targeted Kerberoasting attack and attempt to crack the user's password

Let's see if adunn user hasn any interesting access

**Looking for Interesting Access**

```powershell-session
PS C:\htb> $adunnsid = Convert-NameToSid adunn 
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

<SNIP>
```

`adunn` user has `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-In-Filtered-Set` rights over the domain object adn we can leverage it to perform a DCSync attack.

## Enumerating ACLs with BloodHound

let's look at how much easier this would have been to identify using the extremely powerful BloodHound tool

we can set the `wley` user as our starting node, select the `Node Info` tab and scroll down to `Outbound Control Rights`.

This option will give us objects we have control over directly via group membersip.

If we click on the `1` next to `First Degree Object Control`, we see the first set of rights that we enumerated,

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption><p>ForceChangePassword over damundsen user</p></figcaption></figure>

Right click on the path will give us popup, if we click Help, we will see the following.

* More info on the specific right, tools, and commands that can be used to pull off this attack
* Operational Security (Opsec) considerations
* External references.

<figure><img src="../../../.gitbook/assets/image (1) (1) (2) (1).png" alt=""><figcaption><p>HELP</p></figcaption></figure>

Next, if we click play next to the Transitive Object Control we will see the entire path that we enumerated above.

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption><p>Transitive Object Control</p></figcaption></figure>

Last is to confirm is adunn has DCSync rights.

**Viewing Pre-Build queries through BloodHound**

<figure><img src="../../../.gitbook/assets/image (31) (2).png" alt=""><figcaption></figcaption></figure>

## Assessment

**What is the rights GUID for User-Force-Change-Password?**

* ```powershell-session
  00299570-246d-11d0-a768-00aa006e0529
  ```

**What flag can we use with PowerView to show us the ObjectAceType in a human-readable format during our enumeration?**

* ```
  -ResolveGUIDs
  ```

**What privileges does the user damundsen have over the Help Desk Level 1 group?**

<figure><img src="../../../.gitbook/assets/image (52) (2).png" alt=""><figcaption><p>GenericWrite</p></figcaption></figure>

**Using the skills learned in this section, enumerate the ActiveDirectoryRights that the user forend has over the user dpayne (Dagmar Payne).**

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption><p>GenericAll</p></figcaption></figure>

**What is the ObjectAceType of the first right that the forend user has over the GPO Management group? (two words in the format Word-Word)**







