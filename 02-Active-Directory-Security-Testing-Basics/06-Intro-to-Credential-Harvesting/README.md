# 🔐 Credential Harvesting

> **Purpose:** Structured revision notes for Windows and Active Directory credential harvesting.

---

# 📖 Overview

This module explains where Windows and Active Directory store credential material and demonstrates how credential information can be extracted from:

- LSASS memory
- SAM + SYSTEM hives
- LSA Secrets / cached credentials
- DPAPI Vaults
- NTDS.dit / DRSUAPI

The supplied material then demonstrates how recovered credentials can be cracked or reused to move from local Administrator access toward Domain Admin and ultimately SYSTEM access on the Domain Controller.

---

# 🗂️ Topic Structure

| File | Focus |
|---|---|
| `01-Windows-and-AD-Credential-Stores.md` | Credential stores and what each one contains |
| `02-Credential-Extraction-with-Mimikatz.md` | Mimikatz-based extraction from DPAPI, SAM, LSASS and LSA cache |
| `03-Credential-Harvesting-with-SecretsDump.md` | Remote extraction, DCC2 cracking, DCSync and Pass-the-Hash |
| `04-Credential-Harvesting-Workflow.md` | Complete attack/credential-reuse chain |
| `Tools.md` | Consolidated tool and command reference |

---

# 🔄 Overall Workflow

```text
Local Administrator
        ↓
Credential Stores
        ↓
┌────────────────────────────┐
│ LSASS                      │
│ SAM + SYSTEM               │
│ LSA Cache / Secrets        │
│ DPAPI Vault                │
└────────────────────────────┘
        ↓
Credential Material
        ↓
Offline Cracking / Reuse
        ↓
Higher-Privilege Account
        ↓
Domain Controller
        ↓
DCSync / NTDS Credential Data
        ↓
NTLM Hash
        ↓
Pass-the-Hash
        ↓
SYSTEM
```

---

# 🎯 Learning Objectives

By completing this module, you should be able to:

- Explain the major Windows/AD credential stores.
- Understand what LSASS contains.
- Explain the relationship between SAM and SYSTEM.
- Understand LSA Secrets and cached domain credentials.
- Understand DPAPI and its user-specific protection model.
- Explain the importance of NTDS.dit on a Domain Controller.
- Use the supplied Mimikatz commands in a lab environment.
- Extract local SAM hashes with registry hive copies.
- Understand DCC2 / MSCacheV2 and why it requires offline cracking.
- Use `secretsdump.py` for local and domain credential extraction.
- Understand the DCSync / DRSUAPI concept.
- Understand Pass-the-Hash.
- Follow the complete credential-harvesting chain demonstrated by the lab.

---

# 🧠 Core Concepts

## Credential Stores

```text
LSASS
→ Live authentication material

SAM + SYSTEM
→ Local account hashes

LSA Secrets / Cache
→ Cached domain and service credential material

DPAPI
→ User-protected application secrets

NTDS.dit
→ Domain credential database
```

## Credential Types

```text
NTLM
→ Can potentially be reused through Pass-the-Hash

DCC2 / MSCacheV2
→ Offline cracking

Kerberos Keys / Tickets
→ Kerberos authentication material
```

---

# 🛠️ Main Tools

```text
Mimikatz
secretsdump.py
John the Ripper
Hashcat
psexec.py
reg.exe
Remmina / RDP
```

See [`Tools.md`](Tools.md) for the full command reference.

---

# ⚠️ Lab Scope

The supplied material contains lab-specific credentials, usernames, IP addresses and hashes.

Examples such as:

```text
10.220.10.20
10.220.10.10
TRYHACKME
```

belong to the training environment.

Use credential-harvesting and credential-reuse techniques only against systems for which you have explicit authorisation.

---

# 📚 Source

Based on the supplied **Credential Harvesting** training material.
