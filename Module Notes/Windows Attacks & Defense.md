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