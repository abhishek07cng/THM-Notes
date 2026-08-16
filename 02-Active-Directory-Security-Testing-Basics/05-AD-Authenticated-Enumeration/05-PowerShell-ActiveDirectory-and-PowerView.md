# 🐚 PowerShell ActiveDirectory Module and PowerView

> **Topic:** AD Authenticated Enumeration
>
> **Focus:** Efficient Active Directory enumeration using PowerShell's official ActiveDirectory module and PowerView.

---

# 📖 Overview

The supplied material introduces two PowerShell-based approaches:

1. **ActiveDirectory module**
2. **PowerView** from the PowerSploit framework

These provide more structured Active Directory enumeration than basic CMD commands.

---

# 1️⃣ ActiveDirectory PowerShell Module

The ActiveDirectory module is available on Domain Controllers.

For other Windows systems, the supplied material notes that **Remote Server Administration Tools (RSAT)** may be required.

---

# 🔎 Checking Availability

```powershell
Get-Module -ListAvailable ActiveDirectory
```

Import the module:

```powershell
Import-Module ActiveDirectory
```

---

# 👤 User Enumeration

List all AD users:

```powershell
Get-ADUser -Filter *
```

This can return information such as:

- Distinguished Name
- Enabled state
- Name
- Object GUID
- SAM account name
- SID
- User Principal Name

---

# 🔍 Query a Specific User

```powershell
Get-ADUser -Identity <username>
```

To retrieve all properties:

```powershell
Get-ADUser -Identity <username> -Properties *
```

---

# 🎯 Select Interesting Properties

Example:

```powershell
Get-ADUser -Identity Administrator -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
```

Useful properties mentioned in the source include:

```text
LastLogonDate
MemberOf
Description
Title
PwdLastSet
```

---

# 🔎 Filter Usernames

Find accounts with `admin` in their name:

```powershell
Get-ADUser -Filter "Name -like '*admin*'"
```

---

# 👥 Group Enumeration

List all groups:

```powershell
Get-ADGroup -Filter *
```

Display only group names:

```powershell
Get-ADGroup -Filter * | Select Name
```

---

# 🔐 Group Members

```powershell
Get-ADGroupMember -Identity "Group Name"
```

Examples:

```powershell
Get-ADGroupMember -Identity "Remote Management Users"
```

```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

---

# 🖥️ Computer Enumeration

List all domain computers:

```powershell
Get-ADComputer -Filter *
```

Display computer names and operating systems:

```powershell
Get-ADComputer -Filter * | Select Name, OperatingSystem
```

---

# 🔒 Domain Password Policy

```powershell
Get-ADDefaultDomainPasswordPolicy
```

The supplied lab output demonstrates fields such as:

```text
ComplexityEnabled
LockoutDuration
LockoutObservationWindow
LockoutThreshold
MaxPasswordAge
MinPasswordAge
MinPasswordLength
PasswordHistoryCount
ReversibleEncryptionEnabled
```

---

# 2️⃣ PowerView

## What Is PowerView?

PowerView is a PowerShell-based domain enumeration tool from the PowerSploit framework.

It can perform tasks such as:

- User enumeration
- Group enumeration
- Computer enumeration
- Domain reconnaissance
- Trust discovery

The supplied material compares PowerView to an evolution of commands such as:

```text
net user
net group
```

---

# 📁 PowerSploit Location

The lab provides PowerSploit at:

```text
C:\Users\asrepuser1\Downloads\PowerSploit-master
```

PowerView is located in:

```text
C:\Users\asrepuser1\Downloads\PowerSploit-master\Recon
```

---

# 📥 Import PowerView

Move to the Recon directory and run:

```powershell
Import-Module .\PowerView.ps1
```

---

# 👤 PowerView User Enumeration

List domain users:

```powershell
Get-DomainUser
```

---

# 🔎 Filter Users

Find usernames containing `admin`:

```powershell
Get-DomainUser *admin*
```

This is useful when looking for potentially privileged accounts.

---

# 👥 PowerView Group Enumeration

List domain groups:

```powershell
Get-DomainGroup
```

Filter groups containing `admin`:

```powershell
Get-DomainGroup "*admin*"
```

The source also mentions:

```powershell
Get-NetGroup
```

as an equivalent command.

---

# 🖥️ PowerView Computer Enumeration

List domain computers:

```powershell
Get-DomainComputer
```

The source also identifies:

```powershell
Get-NetComputer
```

as an equivalent.

---

# 👑 Administrative Accounts

PowerView can identify accounts with administrative privileges:

```powershell
Get-DomainUser -AdminCount
```

This returns domain users whose `adminCount` indicates administrative privilege history.

---

# 🎫 Service Principal Names

Find accounts with non-null SPNs:

```powershell
Get-DomainUser -SPN
```

The supplied material notes that these accounts can be considered for Kerberoasting during an authorised assessment.

---

# 🆚 CMD vs PowerView

| Task | CMD | PowerView |
|---|---|---|
| Domain users | `net user /domain` | `Get-DomainUser` |
| User filtering | Limited | `Get-DomainUser *admin*` |
| Domain groups | `net group /domain` | `Get-DomainGroup` |
| Group filtering | Limited | `Get-DomainGroup "*admin*"` |
| Domain computers | `net group "Domain Computers" /domain` | `Get-DomainComputer` |
| Admin accounts | Manual inspection | `Get-DomainUser -AdminCount` |
| SPN accounts | Manual inspection | `Get-DomainUser -SPN` |

---

# 🔄 Recommended Enumeration Flow

```text
Authenticated Shell
       ↓
Import-Module ActiveDirectory
       ↓
Get-ADUser
       ↓
Get-ADGroup
       ↓
Get-ADComputer
       ↓
Password Policy
       ↓
PowerView
       ↓
Get-DomainUser
       ↓
Get-DomainGroup
       ↓
Get-DomainComputer
       ↓
AdminCount / SPN Enumeration
```

---

# 💡 Key Takeaways

- PowerShell provides more structured AD enumeration than basic CMD commands.
- The official ActiveDirectory module provides commands such as `Get-ADUser`, `Get-ADGroup`, and `Get-ADComputer`.
- `Get-ADDefaultDomainPasswordPolicy` reveals domain password-policy settings.
- PowerView is part of PowerSploit.
- PowerView provides flexible domain enumeration and filtering.
- `Get-DomainUser -AdminCount` identifies users with administrative privilege history.
- `Get-DomainUser -SPN` identifies accounts with SPNs that may be relevant to Kerberoasting.
