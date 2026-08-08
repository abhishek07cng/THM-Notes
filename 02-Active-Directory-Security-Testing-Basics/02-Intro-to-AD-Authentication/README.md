# 🔐 Intro to Active Directory Authentication

> **TryHackMe Room:** Intro to AD Authentication  
> **Purpose:** Structured revision notes for Active Directory authentication fundamentals, authentication weaknesses, detection, and mitigation.

---

## 📖 Overview

This module covers the fundamentals of authentication in an Active Directory environment.

The main authentication protocols covered are:

- **NetNTLM / NTLM**
- **Kerberos**

The module progresses from authentication fundamentals into common authentication weaknesses, practical demonstrations, detection, and mitigation.

---

## 📚 Module Contents

| File | Topic |
|---|---|
| `01-Introduction.md` | Introduction to Active Directory authentication |
| `02-Authentication-in-AD.md` | Authentication material, authentication vs authorisation, NTLM and Kerberos |
| `03-NetNTLM-Authentication.md` | NTLM challenge-response authentication |
| `04-Kerberos-Authentication.md` | Kerberos authentication and ticket flow |
| `05-Weaknesses-in-AD-Authentication.md` | Common NTLM, Kerberos, and configuration weaknesses |
| `06-Detection-and-Mitigation.md` | Windows authentication events, detection indicators, and defensive controls |
| `07-Conclusion.md` | Final revision and key concepts |
| `Tools.md` | Tools used throughout this module |

---

# 🎯 Learning Objectives

By the end of this module, you should be able to explain:

- What authentication means in Active Directory
- The difference between authentication and authorisation
- Common forms of authentication material
- NTLM challenge-response authentication
- Kerberos ticket-based authentication
- The role of the KDC
- Authentication Service (AS)
- Ticket Granting Service (TGS)
- Ticket Granting Tickets (TGTs)
- Service Tickets
- Service Principal Names (SPNs)
- The KRBTGT account
- Kerberos credential caches
- Common NTLM authentication weaknesses
- Common Kerberos authentication weaknesses
- Pass-the-Hash
- NTLM Relay
- Kerberoasting
- AS-REP Roasting
- Pass-the-Ticket
- Overpass-the-Hash
- Golden Tickets
- Silver Tickets
- Authentication-related Windows Event IDs
- Basic detection and mitigation techniques

---

# 🔄 Authentication Overview

## NTLM

```text
Client
   ↓
Target Service
   ↓
Challenge
   ↓
Client Response
   ↓
Domain Controller
   ↓
Verification
   ↓
Access
```

## Kerberos

```text
Client
   ↓
KDC
   ↓
TGT
   ↓
TGS
   ↓
Service Ticket
   ↓
Target Service
   ↓
Access
```

---

# 🧩 Core Concepts

### Authentication

> Proves who you are.

### Authorisation

> Determines what you are allowed to access.

### NTLM

> Challenge-response authentication.

### Kerberos

> Ticket-based authentication.

### TGT

> Ticket Granting Ticket used to request service tickets.

### SPN

> Service Principal Name identifying a service instance.

### KRBTGT

> Special AD account associated with protecting Kerberos TGTs.

---

# ⚠️ Authentication Weaknesses

## NTLM

- Weak cryptography
- Pass-the-Hash
- NTLM Relay
- Downgrade attacks
- Lack of mutual authentication

## Kerberos

- Kerberoasting
- AS-REP Roasting
- Pass-the-Ticket
- Overpass-the-Hash
- Golden Ticket
- Silver Ticket

## Configuration

- Weak passwords
- Password spraying
- Misconfigured delegation
- Stale credentials

---

# 🛡️ Detection

Important Windows Security Event IDs:

```text
4624 → Successful Logon
4625 → Failed Logon
4768 → Kerberos TGT Request
4769 → Kerberos Service Ticket Request
4771 → Kerberos Pre-Authentication Failure
```

These events can help identify suspicious authentication activity.

---

# 🔐 Mitigation Highlights

Important defensive controls covered in this module include:

- Reduce NTLM usage where possible
- Protect privileged accounts
- Use the Protected Users group
- Enforce SMB signing
- Enable Extended Protection for Authentication (EPA)
- Use strong service-account passwords
- Use gMSA where appropriate
- Protect the KRBTGT account
- Reset KRBTGT credentials appropriately after suspected compromise
- Monitor authentication events
- Configure account lockout policies

---

# 🧪 Practical Lab Environment

Several practical demonstrations in this module use:

```text
Domain:
thm.loc

Target:
SERVER1.thm.loc

Target IP:
192.168.11.51

Domain Controller:
192.168.11.100
```

> These values belong to the TryHackMe lab environment and should be treated as **lab examples**, not general production configuration.

---

# 🛠️ Tools

The tools used throughout this module are documented separately in:

```text
Tools.md
```

The main tooling includes:

- Impacket
- `smbclient.py`
- `getTGT.py`
- `GetUserSPNs.py`
- `ticketer.py`
- Hashcat
- `klist`

---

# 🧠 Revision Flow

```text
AD Authentication
       ↓
Authentication Material
       ↓
Authentication vs Authorisation
       ↓
NTLM
       ↓
Kerberos
       ↓
Authentication Weaknesses
       ↓
Attack Techniques
       ↓
Detection
       ↓
Mitigation
       ↓
Advanced AD Security
```

---

# 🎯 Recommended Revision Order

Study the files in numerical order:

```text
01 → 02 → 03 → 04 → 05 → 06 → 07
```

Pay particular attention to:

```text
NTLM
Kerberos
TGT
TGS
SPN
KRBTGT
NTLM Hash
ccache
Pass-the-Hash
Kerberoasting
Golden Ticket
Event IDs
```

---

# 📌 Important Reminder

The commands and credentials shown in the practical sections are **TryHackMe lab examples**.

Keep them in the notes because they are useful for revision and understanding the workflow, but do not treat the example credentials, IP addresses, hashes, or domain names as real-world values.

---

# 📚 Reference

- TryHackMe — Intro to AD Authentication
