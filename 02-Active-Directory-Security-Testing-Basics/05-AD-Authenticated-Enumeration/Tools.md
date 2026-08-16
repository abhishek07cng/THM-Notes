# 🛠️ AD Authenticated Enumeration — Tools

> **Purpose:** Consolidated tool reference for the entire AD Authenticated Enumeration module.

---

# 📊 Tool Summary

| Tool | Category | Main Use |
|---|---|---|
| **Rubeus** | Kerberos | AS-REP Roasting and Kerberos enumeration |
| **Impacket / GetNPUsers.py** | Kerberos / AD | Enumerate AS-REP-vulnerable accounts and collect hashes |
| **Hashcat** | Password Cracking | Offline cracking of AS-REP hashes |
| **whoami** | Native Windows | Identify current user, groups and privileges |
| **hostname** | Native Windows | Identify host name |
| **systeminfo** | Native Windows | System and domain information |
| **net** | Native Windows | Users, groups, sessions and network information |
| **quser / query user** | Native Windows | Logged-on users and sessions |
| **tasklist** | Native Windows | Running processes |
| **WMIC** | Native Windows | Service and account enumeration |
| **sc** | Native Windows | Service enumeration and configuration |
| **reg** | Native Windows | Registry enumeration |
| **BloodHound-CE** | AD Graph Analysis | Visualise AD relationships and attack paths |
| **SharpHound** | AD Collection | Collect AD relationship data for BloodHound |
| **BloodHound.py** | AD Collection | Python-based BloodHound data collection |
| **AzureHound** | Cloud AD | Azure Entra ID enumeration |
| **PowerShell ActiveDirectory** | AD Enumeration | Native PowerShell AD queries |
| **PowerView** | AD Enumeration | Advanced PowerShell-based domain enumeration |
| **PowerSploit** | PowerShell Framework | Framework containing PowerView and other modules |

---

# 1️⃣ Rubeus

## Purpose

Windows-based Kerberos security testing and enumeration.

## Topic Use

AS-REP Roasting.

## Example

```cmd
Rubeus.exe asreproast
```

---

# 2️⃣ Impacket — GetNPUsers.py

## Purpose

Enumerates users susceptible to AS-REP Roasting and retrieves AS-REP hashes.

## Example

```bash
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

---

# 3️⃣ Hashcat

## Purpose

Offline password cracking.

## AS-REP Mode

```text
18200
```

## Example

```bash
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

# 4️⃣ whoami

## Purpose

Identify the current Windows identity.

```cmd
whoami
```

Detailed information:

```cmd
whoami /all
```

Useful for:

- SID
- Group memberships
- Privileges

---

# 5️⃣ hostname

```cmd
hostname
```

Returns the computer's hostname.

---

# 6️⃣ systeminfo

```cmd
systeminfo
```

Useful for:

- OS information
- Hotfix information
- Domain/workgroup information

Filters:

```cmd
systeminfo | findstr /B "OS"
```

```cmd
systeminfo | findstr /B "Domain"
```

---

# 7️⃣ NET

## Domain Users

```cmd
net user /domain
```

## User Details

```cmd
net user <username> /domain
```

## Domain Groups

```cmd
net group /domain
```

## Group Members

```cmd
net group "Domain Admins" /domain
```

## Domain Computers

```cmd
net group "Domain Computers" /domain
```

## Local Groups

```cmd
net localgroup
```

## Local Administrators

```cmd
net localgroup administrators
```

## SMB Sessions

```cmd
net session
```

---

# 8️⃣ quser / query user

## Purpose

Enumerates logged-on users and sessions.

```cmd
quser
```

Equivalent:

```cmd
query user
```

---

# 9️⃣ tasklist

## Purpose

Lists running processes.

```cmd
tasklist
```

Verbose:

```cmd
tasklist /V
```

---

# 🔟 WMIC

## Purpose

Enumerates Windows services and their associated accounts.

```cmd
wmic service get Name,StartName
```

---

# 1️⃣1️⃣ PowerShell WMI

Equivalent service query:

```powershell
Get-WmiObject Win32_Service | select Name, StartName
```

---

# 1️⃣2️⃣ SC

## List Services

```cmd
sc query state= all
```

## Filter

```cmd
sc query state= all | find "DHCP"
```

## Query Configuration

```cmd
sc qc DHCP
```

---

# 1️⃣3️⃣ REG

## Auto-Logon Configuration

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUsername
```

## Installed Applications

```cmd
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

## Search Registry

```cmd
reg query HKLM /f "password" /t REG_SZ /s
```

---

# 1️⃣4️⃣ BloodHound

## Purpose

Graph-based Active Directory analysis.

Useful for visualising:

- Users
- Groups
- Computers
- Sessions
- ACLs
- Trusts
- Privilege relationships
- Attack paths

---

# 1️⃣5️⃣ SharpHound

## Purpose

Official BloodHound data collector.

## Example

```cmd
.\SharpHound.exe --CollectionMethods All --Domain tryhackme.loc --ExcludeDCs
```

---

# 1️⃣6️⃣ BloodHound.py

## Purpose

Python-based BloodHound collector.

## Example

```bash
bloodhound-python -u asrepuser1 -p qwerty123! -d tryhackme.loc -ns 10.211.12.10 -c All --zip
```

---

# 1️⃣7️⃣ AzureHound

## Purpose

Enumerates Azure Entra ID environments and supports hybrid identity enumeration.

The supplied material mentions:

```text
AzureHound.ps1
```

---

# 1️⃣8️⃣ PowerShell ActiveDirectory Module

## Check Availability

```powershell
Get-Module -ListAvailable ActiveDirectory
```

## Import

```powershell
Import-Module ActiveDirectory
```

## Users

```powershell
Get-ADUser -Filter *
```

## Specific User

```powershell
Get-ADUser -Identity <username>
```

## User Properties

```powershell
Get-ADUser -Identity <username> -Properties *
```

## Groups

```powershell
Get-ADGroup -Filter *
```

## Group Members

```powershell
Get-ADGroupMember -Identity "Group Name"
```

## Computers

```powershell
Get-ADComputer -Filter *
```

## Password Policy

```powershell
Get-ADDefaultDomainPasswordPolicy
```

---

# 1️⃣9️⃣ PowerView

## Purpose

PowerShell-based AD enumeration from the PowerSploit framework.

## Import

```powershell
Import-Module .\PowerView.ps1
```

## Users

```powershell
Get-DomainUser
```

## Filter Users

```powershell
Get-DomainUser *admin*
```

## Groups

```powershell
Get-DomainGroup
```

## Filter Groups

```powershell
Get-DomainGroup "*admin*"
```

## Computers

```powershell
Get-DomainComputer
```

## AdminCount

```powershell
Get-DomainUser -AdminCount
```

## SPNs

```powershell
Get-DomainUser -SPN
```

---

# 🔄 Tool Workflow

```text
Rubeus / GetNPUsers.py
        ↓
AS-REP Roasting
        ↓
Hashcat
        ↓
Authenticated Access
        ↓
whoami / systeminfo / hostname
        ↓
net / quser / tasklist
        ↓
WMIC / sc
        ↓
Registry
        ↓
SharpHound / BloodHound.py
        ↓
BloodHound-CE
        ↓
ActiveDirectory Module
        ↓
PowerView
```

---

# 📌 Quick Revision

```text
Rubeus
→ AS-REP Roasting

GetNPUsers.py
→ AS-REP hash collection

Hashcat
→ Offline cracking

whoami
→ Identity + privileges

net
→ Users + groups + sessions

quser
→ Logged-on users

tasklist
→ Processes

WMIC / sc
→ Services + service accounts

reg
→ Registry

SharpHound
→ BloodHound data collection

BloodHound
→ Graph analysis + attack paths

ActiveDirectory
→ PowerShell AD enumeration

PowerView
→ Advanced AD enumeration
```

---

# ⚠️ Authorisation Reminder

These commands and techniques are intended for authorised security assessments and lab environments.
