# 🖥️ Manual Windows Enumeration

> **Topic:** AD Authenticated Enumeration
>
> **Focus:** Native CMD and PowerShell enumeration after obtaining an authenticated Windows shell.

---

# 📖 Overview

Manual enumeration focuses on understanding the Windows machine and the authenticated user's position in the environment.

The supplied material covers:

- Current user
- Domain membership
- Groups
- Token privileges
- Host information
- Environment variables
- Domain users
- Domain groups
- Local groups
- Logged-on users
- Running processes
- Services
- Service accounts
- Registry information
- Installed applications

Because the techniques rely heavily on native Windows functionality, the material refers to this as:

> **Living off the Land (LOTL)**

---

# 🔑 Initial Access Context

The supplied lab connects to the workstation over SSH:

```bash
ssh asrepuser1@10.211.12.20
```

Lab credentials supplied by the source:

```text
Username: asrepuser1
Password: qwerty123!
```

After login:

```text
tryhackme\asrepuser1@WRK C:\Users\asrepuser1>
```

This shows:

```text
Domain: tryhackme
User: asrepuser1
Hostname: WRK
```

---

# 👤 Who Am I?

## whoami

```cmd
whoami
```

Example:

```text
tryhackme\asrepuser1
```

This identifies the current user and domain.

### Domain Account

```text
DomainName\DomainUser
```

### Local Account

```text
ComputerName\LocalUser
```

---

# 🔍 whoami /all

For detailed account information:

```cmd
whoami /all
```

This displays:

- User SID
- Group memberships
- Account privileges

---

# 🪪 Security Identifier

Example:

```text
tryhackme\asrepuser1
S-1-5-21-1966530601-3185510712-10604624-1641
```

The SID helps identify the security principal.

---

# 👥 Group Memberships

`whoami /all` can reveal membership in groups such as:

```text
BUILTIN\Users
Domain Users
Administrators
Backup Operators
Domain Admins
```

Group membership provides important context about what the current account can access.

---

# ⚡ Important Privileges

The supplied material highlights:

| Privilege | Significance |
|---|---|
| `SeImpersonatePrivilege` | Can enable token impersonation attacks |
| `SeAssignPrimaryTokenPrivilege` | Allows assigning another user's primary token to a process |
| `SeBackupPrivilege` | Can allow reading files while bypassing normal file permissions |
| `SeRestorePrivilege` | Can allow writing files or registry keys while bypassing normal permissions |
| `SeDebugPrivilege` | Can allow attaching to other processes and potentially accessing privileged process memory |

> **Revision Note:** These privileges are especially important during privilege-escalation analysis.

---

# 🖥️ System and Domain Information

Three basic commands:

```cmd
hostname
systeminfo
set
```

---

# 1. hostname

```cmd
hostname
```

Returns the computer's hostname.

Names may provide hints about the system's role.

For example:

```text
DC
PC01
WRK
```

---

# 2. systeminfo

```cmd
systeminfo
```

The supplied material notes that this requires administrator privileges.

It can reveal:

- Windows version
- Installed hotfixes
- Domain/workgroup information
- System configuration

Filter the output:

```cmd
systeminfo | findstr /B "OS"
```

```cmd
systeminfo | findstr /B "Domain"
```

---

# 3. Environment Variables

```cmd
set
```

Environment variables can reveal:

- User directories
- Domain information
- Installed software hints
- Development environment information

For example:

```text
USERDOMAIN
JAVA_HOME
```

In PowerShell:

```powershell
Get-ChildItem Env:
```

or:

```powershell
dir env:
```

---

# 👥 Domain User Enumeration

## List Domain Users

```cmd
net user /domain
```

This queries the Domain Controller for domain user accounts.

The supplied lab output includes accounts such as:

```text
Administrator
asrepuser1
barbara.jones
daniel.turner
danielle.ali
danielle.lee
[...]
```

---

# 🔎 Detailed User Information

```cmd
net user <username> /domain
```

Example:

```cmd
net user daniel.turner /domain
```

Information can include:

- Full name
- Account status
- Account expiration
- Password information
- Last logon
- Group memberships
- Logon hours

---

# 👥 Domain Group Enumeration

List domain groups:

```cmd
net group /domain
```

Interesting groups can include:

```text
Domain Admins
Enterprise Admins
Server Operators
Backup Operators
SQL Admins
```

A group containing `Admin` in its name is worth investigating during authorised enumeration.

---

# 🖥️ Domain Computer Enumeration

Machine accounts can be listed through:

```cmd
net group "Domain Computers" /domain
```

Machine accounts commonly end with:

```text
$
```

Example:

```text
DESKTOP-ACCT05$
```

---

# 👑 Domain Group Membership

Example:

```cmd
net group "Domain Admins" /domain
```

This lists accounts belonging to the selected domain group.

---

# 🏠 Local Group Enumeration

List local groups:

```cmd
net localgroup
```

Example groups include:

```text
Administrators
Backup Operators
Remote Desktop Users
Remote Management Users
Users
```

---

# 🔐 Local Administrators

To list members:

```cmd
net localgroup administrators
```

Example:

```text
Administrator
TRYHACKME\Domain Admins
TRYHACKME\katie.thomas
```

This is useful for determining which domain accounts have local administrative access.

---

# 👤 Logged-On Users

Use:

```cmd
quser
```

or:

```cmd
query user
```

Example:

```text
USERNAME       SESSIONNAME   ID  STATE   IDLE TIME  LOGON TIME
strategos      console       1   Active  2          16/05/2025
administrator  rdp-tcp#0     2   Active  4          16/05/2025
administrator  rdp-tcp#1     3   Active  .          16/05/2025
```

This reveals:

- Username
- Session type
- Session ID
- State
- Idle time
- Logon time

---

# ⚙️ Running Processes

```cmd
tasklist
```

Verbose output:

```cmd
tasklist /V
```

The supplied example includes:

```text
lsass.exe
services.exe
winlogon.exe
```

Running processes can provide useful context about the system and active users/services.

---

# 🌐 SMB Sessions

```cmd
net session
```

This lists SMB sessions between the local computer and other systems.

> **Requirement:** Administrator privileges.

---

# 📁 Previously Logged-On Users

Check:

```text
C:\Users\
```

User profile directories can indicate which users have logged onto the machine previously.

---

# 💡 Key Takeaways

- Start authenticated enumeration by identifying the current user.
- `whoami /all` is one of the most valuable native commands.
- Always inspect groups and privileges.
- `hostname`, `systeminfo`, and environment variables reveal system context.
- `net user /domain` enumerates domain users.
- `net group /domain` enumerates domain groups.
- `net localgroup` enumerates local groups.
- `quser` identifies currently logged-on users.
- `tasklist` reveals running processes.
- `net session` shows SMB sessions.
