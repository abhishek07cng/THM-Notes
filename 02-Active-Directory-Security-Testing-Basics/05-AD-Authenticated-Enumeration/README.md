# 🔐 AD Authenticated Enumeration

> **Purpose:** Structured revision notes for Active Directory enumeration after obtaining authenticated access.

---

# 📖 Overview

Authenticated enumeration provides deeper visibility into an Active Directory environment.

This module covers:

- AS-REP Roasting
- Manual Windows enumeration
- Service accounts
- Environment variables and Registry
- BloodHound / SharpHound
- ActiveDirectory PowerShell module
- PowerView

---

# 🗂️ Topic Structure

| File | Focus |
|---|---|
| `01-AS-REP-Roasting.md` | Pre-authentication-disabled accounts and offline AS-REP cracking |
| `02-Manual-Windows-Enumeration.md` | Native CMD enumeration of users, groups, privileges and sessions |
| `03-Service-Accounts-Environment-and-Registry.md` | Services, service accounts, environment variables and Registry |
| `04-BloodHound-and-SharpHound.md` | Graph-based AD enumeration and attack-path analysis |
| `05-PowerShell-ActiveDirectory-and-PowerView.md` | PowerShell-based AD enumeration |

---

# 🔄 Overall Workflow

```text
Authenticated Access
        ↓
AS-REP Roasting Checks
        ↓
Current User + Privileges
        ↓
System + Domain Information
        ↓
Users + Groups
        ↓
Sessions + Processes
        ↓
Service Accounts
        ↓
Environment + Registry
        ↓
BloodHound Collection
        ↓
PowerShell AD Enumeration
        ↓
PowerView
        ↓
Admin / SPN / Relationship Analysis
```

---

# 🎯 Learning Objectives

By completing this topic, you should be able to:

- Understand AS-REP Roasting.
- Identify accounts with disabled Kerberos pre-authentication.
- Collect AS-REP hashes using the supplied lab tools.
- Crack AS-REP hashes offline with Hashcat.
- Identify your current Windows identity.
- Enumerate privileges and group memberships.
- Discover domain users and groups.
- Identify logged-on users and sessions.
- Enumerate services and service accounts.
- Inspect environment variables and Registry locations.
- Understand BloodHound's graph-based model.
- Collect AD data with SharpHound or BloodHound.py.
- Use BloodHound-CE to analyse relationships.
- Use the ActiveDirectory PowerShell module.
- Use PowerView for flexible AD enumeration.

---

# 🛠️ Tools

See [`Tools.md`](Tools.md) for the complete module-level tool list.

---

# ⚠️ Lab Scope

The commands, IP addresses, usernames, domains and credentials shown in these notes are examples from the supplied TryHackMe material.

Examples such as:

```text
tryhackme.loc
10.211.12.10
10.211.12.20
```

are lab-specific.

Use these techniques only against systems for which you have explicit authorisation.

---

# 📚 Reference

- TryHackMe — AD: Authenticated Enumeration
