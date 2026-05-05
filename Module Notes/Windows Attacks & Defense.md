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









