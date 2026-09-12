# Active Directory home lab

Built a working Windows domain from scratch in a virtual lab: a Windows Server 2025 domain controller with AD DS, DNS, and DHCP, and a Windows 11 client joined to it. This is the hands-on version of the tasks a help desk or desktop support role does every day: managing accounts, resolving DNS, applying policy, and troubleshooting when it breaks.

## At a glance

| | |
|---|---|
| **Platform** | Oracle VirtualBox 7.2 |
| **Server** | Windows Server 2025 (DC01) — AD DS, DNS, DHCP |
| **Client** | Windows 11 (CLIENT01), domain-joined |
| **Domain** | lab.local |
| **Outcome** | A functioning domain with users, groups, a password policy, and a joined client, verified end to end |

## What this demonstrates

- Standing up a domain controller and a new forest
- DNS zones and a forwarder for external name resolution
- Creating OUs, security groups, and user accounts
- Applying a domain password policy with Group Policy
- Building and domain-joining a Windows 11 client
- Administering and verifying Active Directory with PowerShell
- Working through real failures: VM boot, DNS, credentials, and syntax

> Network addresses are on a private lab subnet. Public addresses are not shown.

## Build walkthrough

### 1. Domain controller and DNS

Promoted DC01 to a domain controller for a new forest, `lab.local`. DNS installs with the promotion, so the forward and reverse lookup zones came up with it.

![DNS Manager showing zones on DC01](images/01-dns-manager.png)

### 2. DNS forwarder and external resolution

Added a forwarder to the local router so the DC resolves names outside the domain, then confirmed it with an external lookup.

![Editing the DNS forwarder](images/02-dns-forwarder.png)
![nslookup resolving an external domain](images/03-nslookup-external.png)

### 3. Organizational Units

Created a `Users (OU)` for accounts and a `Groups` OU for security groups, kept separate from the built-in containers so Group Policy can target them.

![OU structure in ADUC](images/04-ou-structure.png)

### 4. User accounts

Created users in the Users OU, each set to change password at next logon, mirroring real onboarding.

![Creating a user](images/05-create-user.png)

### 5. Security groups and membership

Created two security groups, `HR-Team` and `IT-Helpdesk`, and added the right users to each.

![Adding a user to a group](images/06-add-to-group.png)
![Group membership](images/07-group-membership.png)

### 6. Group Policy: password policy

Set the domain password policy: minimum length 12, complexity required, 24 passwords remembered, maximum age 42 days, minimum age 1 day.

![Password policy settings](images/08-password-policy.png)
![Complexity enabled](images/09-password-complexity.png)

### 7. PowerShell administration

Queried accounts and confirmed group membership with the Active Directory module.

```powershell
Import-Module ActiveDirectory
Get-ADUser jdoe
Get-ADGroupMember "HR-Team"
Get-ADGroupMember "IT-Helpdesk"
```

![PowerShell verification](images/10-powershell-verify.png)

### 8. Windows 11 client and domain join

Built a Windows 11 client (EFI, TPM 2.0, Secure Boot), pointed its DNS at the domain controller, and joined `lab.local`.

![Client DNS set to the DC](images/11-client-dns.png)
![Joining the domain](images/12-domain-join.png)
![Welcome to the lab.local domain](images/13-welcome-domain.png)

Logged in as a domain user and confirmed the account:

![Domain user logged in](images/14-domain-login.png)
![whoami showing the domain account](images/15-whoami.png)

The client then appears as a computer object on the domain controller:

![CLIENT01 in ADUC](images/16-computer-object.png)

## Verification

- Internal and external name resolution confirmed with `nslookup`
- Users and group membership confirmed with `Get-ADUser` and `Get-ADGroupMember`
- Password policy visible in the Group Policy editor
- Client joined to `lab.local`, logged in as a domain user (`whoami` returns `lab\asmith`), and registered as a computer object on the DC

## Troubleshooting

**Windows 11 client would not boot the installer.** The VM hit "no bootable option or device was found." Windows 11 media needs EFI, plus TPM 2.0 and Secure Boot to pass the install checks. Enabling those let it boot and install.

![VM failed to boot](images/tshoot-boot-failure.png)
![VM configured with EFI, TPM 2.0, and Secure Boot](images/tshoot-vm-config.png)

**Client could not resolve the domain.** Its DNS server was mistyped (`198.x` instead of the DC's `192.x`), so lookups timed out. Correcting it to the DC's address fixed resolution and the join completed.

![Mistyped DNS address](images/tshoot-dns-typo.png)

**Join rejected the credentials.** The account needs the domain-qualified format. `LAB\Administrator` instead of a bare username completed the join.

**`Import-Module Active Directory` failed.** The module name has no space. `Import-Module ActiveDirectory` loaded it correctly.

## Next steps

- Link a GPO to the Users OU and confirm it applies on the client with `gpresult`
- Bulk-create users from a CSV with PowerShell
- Connect the domain to Microsoft Entra ID for hybrid identity
