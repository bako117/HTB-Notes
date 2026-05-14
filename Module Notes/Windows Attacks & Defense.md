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

# Credentials in Object Properties

Each object in AD contains properties, for ex: a `user` object may contain: 
- Is the account active
- When does the account expire
- When was the last password change
- What is the name of the account
- Office location for the employee and phone number

A common practice in the past was to add the user's (or service account's) password in the `Description` or `Info` properties, thinking that administrative rights in AD are needed to view these properties. However, `every` domain user can read most properties of an object making this a viable/simple attack method. You can use powershell to query this. 

## Prevention and Detection
The best way to prevent this is to create a baseline for user activity and find any anomalous actions. This can be tricky. Additionally, to prevent this behavior, a regular audit of object properties to ensure no credentials are stored is important. 

# DCSync

DCSync is an attack designed to impersonate a domain controller to extract password hashes from Active Directory. A user or computer just needs these two permissions: 
- `Replicating Directory Changes`
- `Replicating Directory Changes All`

To run the attack: 
- Start a command shell as someone with the privileges
- Run mimikatz and specify the domain and user we want or we can specify all and get every hash. 
	- It is possible to specify the `/all` parameter instead of a specific username, which will dump the hashes of the entire AD environment. We can perform `pass-the-hash` with the obtained hash and authenticate against any Domain Controller.
- Then we get the hash!
```cmd-session
Credentials:
  Hash NTLM: fcdc65703dd2b0bd789977f1f3eeaecf
```


## Prevention and Detection

Domain replication is actually a really common activity that occurs between domain controllers. It is unfair to block it out of the bo0x, that would break AD. To prevent this type of attack we should utilize a RPC firewall designed to help us block any dcsync requests form uknown hosts or accounts. 

Detecting DCSync is easy because each Domain Controller replication generates an event with the ID `4662`. We can pick up abnormal requests immediately by monitoring for this event ID and checking whether the initiator account is a Domain Controller.Because replications frequently occur, your detections should look for ther following properties: 
- Either the property `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` or `1131f6ad-9c07-11d1-f79f-00c04fc2dcd2` is [present in the event](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/1522b774-6464-41a3-87a5-1e5633c3fbbb).
- Whitelisting systems/accounts with a (valid) business reason for replicating, such as `Azure AD Connect` (this service constantly replicates Domain Controllers and sends the obtained password hashes to Azure AD).

# Golden Ticket
The `Kerberos Golden Ticket` is an attack in which attackers can create/generate tickets for any user in the Domain, therefore effectively acting as a Domain Controller.

At the start of a domain, a unique user is created, `krbtgt`. It is a disabled user account that cannot be edited, deleted,  renamed, controlled.  The Domain Controller's KDC service will use the password of `krbtgt` to derive a key with which it signs all Kerberos tickets. This password's hash is the most trusted object in the entire Domain because it is how objects guarantee that the environment's Domain issued Kerberos tickets. 

Therefore, `any user` possessing the password's hash of `krbtgt` can create valid Kerberos TGTs.

## Attack

To perform the `Golden Ticket` attack, we can use `Mimikatz` with the following arguments:

- `/domain`: The domain's name.
- `/sid`: The domain's SID value.
- `/rc4`: The password's hash of `krbtgt`.
- `/user`: The username for which `Mimikatz` will issue the ticket (Windows 2019 blocks tickets if they are for inexistent users.)
- `/id`: Relative ID (last part of `SID`) for the user for whom `Mimikatz` will issue the ticket.

Additionally, advanced attackers mostly will specify values for the `/renewmax` and `/endin` arguments, as otherwise, `Mimikatz` will generate the ticket(s) with a lifetime of 10 years, making it very easy to detect by EDRs:

First we need to get the hash of the krbtgt user and the domain SID value. We can use the previous DCSync technique to get the hash. 

```cmd-session
mimikatz # lsadump::dcsync /domain:eagle.local /user:krbtgt
```

We will use the `Get-DomainSID` function from [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) to obtain the SID value of the Domain:

```powershell-session
PS C:\Users\bob\Downloads> . .\PowerView.ps1
PS C:\Users\bob\Downloads> Get-DomainSID
```

Now, armed with all the required information, we can use `Mimikatz` to create a ticket for the account `Administrator`. The `/ptt` argument makes `Mimikatz` [pass the ticket into the current session](https://adsecurity.org/?page_id=1821#KERBEROSPTT):

```cmd-session
mimikatz # kerberos::golden /domain:eagle.local /sid:S-1-5-21-1518138621-4282902758-752445584 /rc4:db0d0630064747072a7da3f7c3b4069e /user:Administrator /id:500 /renewmax:7 /endin:8 /ptt
```
The output shows that `Mimikatz` injected the ticket in the current session, and we can verify that by running the command `klist` (after exiting from `Mimikatz`):

## Detection and Prevention

Preventing the creation of forged tickets is difficult as the `KDC` generates valid tickets using the same procedure. Therefore, once an attacker has all the required information, they can forge a ticket. Nonetheless, there are a few things we can and should do:
- Block privileges users from authenticating to devices
- Periodically reset the `krbtgt` account
-  Enforce SIDHistory to prevent escalation from a child domain to a parent domain

Establishing baselines for users is the best way to detect anomalies. For ex: looking at privledged accoutns and seeing one authenticate to a non-paw device is bad!!

# Kerberos Constrained Delegation

Kerberos delegation lets a trusted service **impersonate a user to another service without knowing the user’s password**.

The simplest way to think about it:

> Normal Kerberos: “I am bako. Let me access the web app.”  
> Delegation: “The web app is allowed to use bako’s identity to access the database for him.”

## Basic example

You log into a company web app:

```
User → Web Server → SQL Server
```

The web server needs to query SQL for your data.

Without delegation, SQL only sees:

```
WebServer$ is asking for data
```

With delegation, SQL can see:

```
bako is asking for data through WebServer$
```

So delegation lets the **middle service** pass your identity to the **backend service**.

There are 3 type of delegations in AD: 
- Unconstrained (Terrible)
- Constrained
- Resource-Based

It is rare to see `Resource-based delegation` configured by an Administrator in production environments ( threat agents often abuse it to compromise devices). However, `Unconstrained` and `Constrained` delegations are commonly encountered in production environments.

## Attack
First create a NTLM hash for the compromised user account with the new password we have: 
```powershell-session
PS C:\Users\bob\Downloads> .\Rubeus.exe hash /password:Slavi123
```

Then use that to generate tickets for another account: 
```powershell-session
PS C:\Users\bob\Downloads> .\Rubeus.exe s4u /user:webservice /rc4:FCDC65703DD2B0BD789977F1F3EEAECF /domain:eagle.local /impersonateuser:Administrator /msdsspn:"http/dc1" /dc:dc1.eagle.local /ptt
```

The /ptt argument injects the tickets into our session. Then we can do the final command to get to the dir that contains the flag. 
```powershell-session
Enter-PSSession dc1
```




# Printer Spooler & NTLM Relaying 


1. Relay the connection to another DC and perform DCSync (if `SMB Signing` is disabled).
2. Force the Domain Controller to connect to a machine configured for `Unconstrained Delegation` (`UD`) - this will cache the TGT in the memory of the UD server, which can be captured/exported with tools like `Rubeus` and `Mimikatz`.
3. Relay the connection to `Active Directory Certificate Services` to obtain a certificate for the Domain Controller. Threat agents can then use the certificate on-demand to authenticate and pretend to be the Domain Controller (e.g., DCSync).
4. Relay the connection to configure `Resource-Based Kerberos Delegation` for the relayed machine. We can then abuse the delegation to authenticate as any Administrator to that machine.
We can do this by using imnpacket to relay the NTLM hash onwards. 

To prevent this disabled the spool service on DC's using the registry editor. 

# Object ACLs

In AD, Object ACLs are tables or lists that define trustees who have access to a specific object and the level of access.

To identify potential abusable ACLs, we will use [BloodHound](https://github.com/BloodHoundAD/BloodHound) to graph the relationships between the objects and [SharpHound](https://github.com/BloodHoundAD/SharpHound) to scan the environment and pass `All` to the `-c` parameter (short version of `CollectionMethod`):
```powershell-session
PS C:\Users\bob\Downloads> .\SharpHound.exe -c All
```

![[Pasted image 20260510204018.png]]
Bob has full rights over the user Anni and the computer Server01. Below is what Bob can do with each of these:
1. Case 1: Full rights over the user Anni. In this case, Bob can modify the object Anni by specifying some bonus SPN value and then perform the Kerberoast attack against it (if you recall, the success of this attack depends on the password's strength). However, Bob can also modify the password of the user Anni and then log in as that account, therefore, directly inheriting and being able to perform everything that Anni can (if Anni is a Domain admin, then Bob would have the same rights).
2. 1. Case 2: Full control over a computer object can also be fruitful. If `LAPS` is used in the environment, then Bob can obtain the password stored in the attributes and authenticate as the local Administrator account to this server. Another escalation path is abusing `Resource-Based Kerberos Delegation`, allowing Bob to authenticate as anyone to this server. Recall that from the previous attack, Server01 is trusted for Unconstrained delegation, so if Bob was to get administrative rights on this server, he has a potential escalation path to compromise the identity of a Domain Controller or other sensitive computer object.

# PKI 
After `SpectreOps` released the research paper [Certified Pre-Owned](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf), `Active Directory Certificate Services` (`AD CS`) became one of the most favorite attack vectors for threat agents due to many reasons, including:

1. Using certificates for authentication has more advantages than regular username/password credentials.
2. Most PKI servers were misconfigured/vulnerable to at least one of the eight attacks discovered by SpectreOps (various researchers have discovered more attacks since then).

There are a plethora of advantages to using certificates and compromising the `Certificate Authority` (`CA`):
- Users and machines certificates are valid for 1+ years.
- Resetting a user password does not invalidate the certificate. With certificates, it doesn't matter how many times a user changes their password; the certificate will still be valid (unless expired or revoked).
- Misconfigured templates allow for obtaining a certificate for any user.
- Compromising the CA's private key results in forging `Golden Certificates`.



