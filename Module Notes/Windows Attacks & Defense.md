AD is a distributed, hierarchical structure that allows centralized management of an organization's resources, including users, computers, groups, network devices and file shares, group policies, devices, and trusts.

AD provides authentication, accounting, and authorization functionalities within a Windows enterprise environment. It also allows administrators to manage permissions and access to network resources.

# Refresher
A `domain` is a group of objects that share the same AD database, such as users or devices.

A `tree` is one or more domains grouped. Think of this as the domains `test.local`, `staging.test.local`, and `preprod.test.local`, which will be in the same tree under `test.local`. Multiple trees can exist in this notation.

A `forest` is a group of multiple trees. This is the topmost level, which is composed of all domains.

`Organizational Units` (`OU`) are Active Directory containers containing user groups, Computers, and other OUs.

`LDAP` is a protocol that systems in the network environment use to communicate with Active Directory. Domain Controller(s) run LDAP and constantly listen for requests from the network.

`Authentication` in Windows Environments:
- Username/Password, stored or transmitted as password hashes (`LM`, `NTLM`, `NetNTLMv1`/`NetNTLMv2`).
- `Kerberos` tickets (Microsoft's implementation of the Kerberos protocol). Kerberos acts as a trusted third party, working with a domain controller (DC) to authenticate clients trying to access services. The Kerberos authentication workflow revolves around tickets that serve as cryptographic proof of identity that clients exchange between each other, services, and the DC.
- `Authentication over LDAP`. Authentication is allowed via the traditional username/password or user or computer certificates.

# Kerberoasting

Kerberoasting abuses the idea of service accounts in an AD network. You request tickets for services, and then go try to crack them offline to pivot. 

To obtain crackable tickets, we can use Rubeus, when specifying the right options, it will extract tickets for every account that has an SPN registered. It works very simply

First an LDAP query for every user account with an SPN
Then request a ticket for each given SPN

After getting our tickets, we can use tools like hashcat to try and break the tickets. If we crack them, we can authenticate as the service. Nice!

## Prevention 
The success of this attack is determined based some factors. For starters, we need SPN's with weak passwords to be able to crack. Limiting the # of active SPNs and making super long random passwords will help here. 

Additionally GMSA accounts are really helpful. They are service account tied to a signle server instance automatically managed by AD with the longest possible password automatically rotating. Not every service supports these but we should use them when possible.

## Detection 
`Event 4769` represents the requesting of a service account. However this event alone generates massive amounts of volume as environments scale. We can take data from this and build a detection to help us find potential Kerberoasting activity. 

1. Look for oddities in the type of encryption used for the tickets
2. Look for users requesting high volumes of tickets
3. Utilize a **honeypot*** user to identify if someone is breaking in


# AS-REProasting

The `AS-REProasting` attack is similar to the `Kerberoasting` attack; we can obtain crackable hashes for user accounts that have the property `Do not require Kerberos preauthentication`, enabled.

Using `Rubeus` with the asreproast action will find the hashes for each user account that has this property. 

Then we will use `hashcat` to break the hahses again. For `hashcat` to be able to recognize the hash, we need to edit it by adding `23$` after `$krb5asrep$`

## Prevention

This attack is only successful when a weak user password is combined with ``Do not require Kerberos preauthentication`` property. To prevent this we should limit the amount of accounts that have this property, accounts that do require it should be reviewed at least quarterly. Accounts that require this should also have passwords at least 20 characters long to thwart cracking attempts.

## Detection

We can detect this type of behavior by looking at event code 4768, signaling that a kerberos authentication ticket has been generated. However, the amount of these are so ridiculously high that we would only want to search for a field for preauthentication type and set it equal to 0. Additionally we would want to look for weaker encryption algorithms that Rubeus defaults to, like RC4. 

Honeypot users are another good example of a way we can detect this type of attack. 

# GPP Passwords

`SYSVOL` is a network share on all Domain Controllers, it contains logon scripts, group policy data, and other required domain wide data.

AD stores all of its group policies in `\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\` in 2008, lol obama. Group Policy Preferences (GPP) was released. You could store credentials within this volume for use later on. 

During engagements, we might encounter scheduled tasks and scripts executed under a particular user and contain the username and an encrypted version of the password in XML policy files. The encryption key that AD uses to encrypt the XML policy files (the `same` for all Active Directory environments) was released on Microsoft Docs, allowing anyone to decrypt credentials stored in the policy files. Anyone can decrypt the credentials because the `SYSVOL` folder is accessible to all 'Authenticated Users' in the domain, which includes users and computers. Microsoft published the [AES private key on MSDN](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN):

![[Pasted image 20260505211636.png]]

To abuse `GPP Passwords`, we will use the [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1) function from `PowerSploit`, which automatically parses all XML files in the Policies folder in `SYSVOL`, picking up those with the `cpassword` property and decrypting them once detected:

## Prevention
Once the encryption key was made public and started to become abused, Microsoft released a patch (`KB2962486)` in 2014 to prevent `caching credentials` in GPP. Therefore, GPP should no longer store passwords in new patched environments. However, older passwords are still cached within the environment. 

## Detection

there are two methods for detecting this attack

1. If we are auditing file access, create a dummy XML file with a cpassword inside it, then any tool that tries to read it is likely very suspicious. Like a honeypot xml file to detect on. 
2. Create a honey pot user or look for authentications for the user's with these credentials stored if necessary. 

## Lab 

```PS
Set-ExecutionPolicy Unrestricted -Scope CurrentUser
```

# GPO Permissions/GPO Files

A [Group Policy Object (GPO)](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects) is a virtual collection of policy settings that has a unique name like a GUID. Each GPO contains a collection of zero or more policy settings.

They are linked to OU's in the AD environmaent to apply setting to objects that reside in the OU.  GPOs can be restricted to which objects they apply by specifying, for example, an AD group (by default, it applies to Authenticated Users) or a WMI filter (e.g., apply only to Windows 10 machines).

When we create a new GPO, only Domain admins (and similar privileged roles) can modify it. However, within environments, we will encounter different delegations that allow less privileged accounts to perform edits on the GPOs; this is where the problem lies.

If a compromised user account can edit a GPO, then they can maliciously edit the GPO to run a file or startup script on every host within the vulnerable GPO. 

If a user can reach specific network shares, a file replacement could derail a benign GPO into running a malicious file. 

## Prevention
To prevent this kind of behavior, we need top lock down who can actually edit and create GPO's as much as possible. It is easy to detect a modified GPO, if the activity is unexpected than we know it could be bad. From a defensive point of view, if a user who is `not` expected to have the right to modify a GPO suddenly appears here, then a red flag should be raised.


# Credentials In Shares

Credentials can lie exposed in Network shares and be a pivot point for attackers after initial access. Creds often lie in these typs of files:
- Batch (.bat)
- CMD
- PowerShell (.ps1)
- .conf
- .ini
- config files

It is the equivalent of leaving a .txt file on your machine with passwords or a post-it on your screen, but worse. To attack this, first we use powerview's `Invoke-ShareFinder` function. It will allow us to find shares our user account has access to. By removing the $c and $ipc share, 

```powershell-session
PS C:\Users\bob\Downloads> Invoke-ShareFinder -domain eagle.local -ExcludeStandard -CheckShareAccess
```

Now we navigate to our desired share and us e the `findstr` function to sift through the share. 

```powershell-session
PS Microsoft.PowerShell.Core\FileSystem::\\Server01.eagle.local\dev$> findstr /m /s /i "pass" *.bat
```

`.bat`, `.cmd`, `.ps1`, `.conf`, `.config`, and `.ini` are the file types we want. 

- `/s` forces to search the current directory and all subdirectories
- `/i` ignores case in the search term
- `/m` shows only the filename for a file that matches the term

Search terms we might use are things like "pw" "pass" NETBIOS domain name, 

To prevent this, we should lock down file shares as much as possible. Additionally, we should analyze user behavior to determine if this is normal. 

Another detection technique is discovering the `one-to-many` connections, for example, when `Invoke-ShareFinder` scans every domain device to obtain a list of its network shares. It would be abnormal for a workstation to connect to 100s or even 1000s of other devices simultaneously.

