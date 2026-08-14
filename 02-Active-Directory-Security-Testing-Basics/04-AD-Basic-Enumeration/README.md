# 🔎 AD Basic Enumeration

> **TryHackMe Topic:** AD Basic Enumeration
>
> **Purpose:** Structured revision notes for enumerating a Windows Active Directory environment before obtaining valid domain credentials.

---

# 📖 Overview

AD enumeration is a crucial first step in penetration testing Microsoft Windows enterprise networks.

An internal tester may receive VPN access to a target network without user credentials. The goal is therefore to gather information about:

- Users
- Groups
- Computers
- Policies
- Network services
- SMB shares
- Domain structure

This information can reveal potential vulnerabilities and attack paths that may lead to an initial foothold.

---

# 🗂️ Topic Structure

| File | Focus |
|---|---|
| `01-Network-Mapping-and-Host-Discovery.md` | Host discovery and live-system identification |
| `02-Port-Scanning-and-Domain-Controller-Identification.md` | Port scanning and identifying the Domain Controller |
| `03-SMB-Enumeration.md` | SMB shares, permissions and accessible files |
| `04-LDAP-RPC-and-Domain-Enumeration.md` | LDAP anonymous binds, enum4linux-ng and RPC null sessions |
| `05-Kerbrute-Username-Enumeration.md` | Validating AD usernames with Kerbrute |
| `06-Password-Policy-and-Password-Spraying.md` | Password policy enumeration and password spraying |

---

# 🔄 Complete Enumeration Workflow

```text
Target Network
      ↓
Host Discovery
      ↓
Live Hosts
      ↓
Port Scanning
      ↓
Identify Domain Controller
      ↓
SMB Enumeration
      ↓
LDAP Enumeration
      ↓
RPC Enumeration
      ↓
RID Cycling
      ↓
Candidate Usernames
      ↓
Kerbrute
      ↓
Valid AD Users
      ↓
Password Policy
      ↓
Password Spraying
      ↓
Valid Credentials
```

---

# 🛠️ Main Tools

The topic uses or mentions:

```text
fping
Nmap
smbclient
smbmap
enum4linux
enum4linux-ng
ldapsearch
rpcclient
Kerbrute
CrackMapExec
Impacket smbclient
```

See [`Tools.md`](Tools.md) for the consolidated tool reference.

---

# 🎯 Core Learning Goals

By the end of this topic, you should understand how to:

- Discover live hosts in a target subnet.
- Identify services running on Windows systems.
- Recognise likely Domain Controllers.
- Enumerate SMB shares.
- Identify read/write permissions.
- Test anonymous SMB access.
- Query LDAP anonymously when permitted.
- Enumerate users through RPC null sessions.
- Understand RID-based enumeration.
- Validate usernames with Kerbrute.
- Enumerate password policies.
- Perform controlled password spraying in an authorised lab.

---

# ⚠️ Lab and Scope Reminder

All commands, IP addresses, domains, usernames and credentials in these notes are derived from the supplied TryHackMe material.

Examples such as:

```text
10.211.11.10
tryhackme.loc
```

are lab-specific.

Use these techniques only against systems for which you have explicit authorisation.

---

# 📚 Reference

- TryHackMe — AD Basic Enumeration
