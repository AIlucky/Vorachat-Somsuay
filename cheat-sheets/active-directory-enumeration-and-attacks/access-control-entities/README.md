# Access Control Entities

## ACL

ACLs are lists that define

1. who has access to which asset/resource
2. the level of access they are provisioned

The settings themselves in an ACL are called `Access Control Entities` (`ACEs`)

Each **ACE** maps back to a user, group, or process (also known as security principals) and defines the rights granted to that principal

There are two types of ACLs:

1. `Discretionary Access Control List` (`DACL`) -> defines which security principals are granted or denied access to an object.
   1. If a DACL does not exist for an object, all who attempt to access the object are granted full rights
   2. If DACL exists, but does not have ACE specifying specific security settings, the system will deny access to all.
2. `System Access Control Lists` (`SACL`) - allow administrators to log access attempts made to secured objects.

## ACE

ACL contain ACE entries that name a user or group and the level of  access they have.

There are `three` main types of ACEs that can be applied to all securable objects in AD:

<table data-header-hidden><thead><tr><th width="230"></th><th></th></tr></thead><tbody><tr><td><strong>ACE</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Access denied ACE</code></td><td>Used within a DACL to show that a user or group is explicitly denied access to an object</td></tr><tr><td><code>Access allowed ACE</code></td><td>Used within a DACL to show that a user or group is explicitly granted access to an object</td></tr><tr><td><code>System audit ACE</code></td><td>Used within a SACL to generate audit logs when a user or group attempts to access an object. It records whether access was granted or not and what type of access occurred</td></tr></tbody></table>

Each ACE is made up of the following `four` components:

1. The security identifier (SID) of the user/group that has access to the object (or principal name graphically)
2. A flag denoting the type of ACE (access denied, allowed, or system audit ACE)
3. A set of flags that specify whether or not child containers/objects can inherit the given ACE entry from the primary or parent object
4. An [access mask](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN) which is a 32-bit value that defines the rights granted to an object

### Why are ACEs Important?

1. Attackers use ACE to further access or establish persistence
2. Many organizations are unaware of ACEs applied to each object.
3. Many organizations are unaware of incorrectly applying ACEs
4. Auto scans can't detect wrong ACEs
5. ACL abuse can be greate way to move vertically and achive full domain compromise

Some example Active Directory object security permissions are as follows.

* `ForceChangePassword` abused with `Set-DomainUserPassword`
  * `ForceChangePassword` gives us the right to force change password without knowing the password before hand
* `Add Members` abused with `Add-DomainGroupMember`
* `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
  * `GenericAll` Grants full control over a target object. if this is granted over a user or group, we could modify group membership, force change a password, or perform a targeted Kerberoasting attack
* `GenericWrite` abused with `Set-DomainObject`
  * `GenericWrite` If we have this right we could assign SPN and perform kerberoasting attack.
* `WriteOwner` abused with `Set-DomainObjectOwner`
* `WriteDACL` abused with `Add-DomainObjectACL`
* `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
* `Addself` abused with `Add-DomainGroupMember`
  * `Addself` shows security groups that a user can add themselves to.

![Charlie Bromberg (Shutdown)](https://academy.hackthebox.com/storage/modules/143/ACL\_attacks\_graphic.png)

### ACL Attacks in the Wild

We can use ACL attacks for:

* Lateral movement
* Privilege escalation
* Persistence

Some common attack scenarios may include:

<table><thead><tr><th width="198">Attack</th><th>Description</th></tr></thead><tbody><tr><td><code>Abusing forgot password permissions</code></td><td>Help Desk and other IT users are often granted permissions to perform password resets and other privileged tasks. If we can take over an account with these privileges (or an account in a group that confers these privileges on its users), we may be able to perform a password reset for a more privileged account in the domain.</td></tr><tr><td><code>Abusing group membership management</code></td><td>It's also common to see Help Desk and other staff that have the right to add/remove users from a given group. It is always worth enumerating this further, as sometimes we may be able to add an account that we control into a privileged built-in AD group or a group that grants us some sort of interesting privilege.</td></tr><tr><td><code>Excessive user rights</code></td><td>We also commonly see user, computer, and group objects with excessive rights that a client is likely unaware of. This could occur after some sort of software install (Exchange, for example, adds many ACL changes into the environment at install time) or some kind of legacy or accidental configuration that gives a user unintended rights. Sometimes we may take over an account that was given certain rights out of convenience or to solve a nagging problem more quickly.</td></tr></tbody></table>
