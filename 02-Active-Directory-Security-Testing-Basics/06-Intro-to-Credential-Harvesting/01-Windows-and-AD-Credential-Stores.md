# 🔐 Windows & Active Directory Credential Stores

> **Topic:** Credential Harvesting
>
> **Focus:** Understanding where Windows and Active Directory store credential material, why those stores exist, and how they can be accessed during an authorised penetration test.

---

# 📖 Overview

Windows and Active Directory store credentials in different locations depending on:

- Whether the system is standalone or domain-joined.
- Whether credentials are needed for login.
- Re-authentication.
- Offline access.
- Application integration.

For penetration testers, these storage mechanisms represent different credential-collection opportunities, each with different privilege requirements and outputs.

---

# 🗂️ Credential Stores

The major stores covered in this module are:

```text
LSASS Memory
     ↓
SAM + SYSTEM Hives
     ↓
LSA Secrets
     ↓
DPAPI Vault
     ↓
NTDS.dit
```

---

# 1️⃣ LSASS Memory

**LSASS** stands for:

```text
Local Security Authority Subsystem Service
```

It enforces Windows security policies and manages authentication.

LSASS can hold sensitive credential material in memory, including:

- NTLM hashes
- LM password hashes
- Kerberos TGTs
- Kerberos service tickets
- Occasionally plaintext credentials

This supports features such as Single Sign-On (SSO), where credentials can be reused transparently between services.

---

## 🎯 Why LSASS Is Valuable

Because LSASS contains live authentication material, an attacker with sufficient privileges can dump the `lsass.exe` process memory.

This can expose credentials useful for:

- Lateral movement
- Privilege escalation
- Credential reuse

---

## 🛠️ Mimikatz Examples

```text
sekurlsa::logonpasswords
```

```text
sekurlsa::minidump
```

---

# 2️⃣ SAM + SYSTEM Hives

The **Security Accounts Manager (SAM)** stores password hashes for local Windows accounts.

This includes local administrator accounts.

The hashes are encrypted.

To decrypt the SAM contents, the attacker also needs the **BootKey**, which is derived from the SYSTEM hive.

---

## 📍 Locations

SAM:

```text
%SystemRoot%\system32\config\SAM
```

SYSTEM:

```text
%SystemRoot%\system32\config\SYSTEM
```

---

## 🔄 Extraction Concept

```text
SAM Hive
   +
SYSTEM Hive
   ↓
BootKey / System Key
   ↓
Recover Local Account Hashes
   ↓
Offline Cracking / Pass-the-Hash
```

The supplied material notes that extraction typically requires SYSTEM privileges, for example through registry export, Volume Shadow Copy, or direct offline file access.

---

## 🛠️ Mimikatz

```text
lsadump::sam
```

---

# 3️⃣ LSA Secrets

LSA Secrets are stored under:

```text
HKLM\SECURITY\Policy\Secrets
```

They can contain sensitive information such as:

- Cached domain credentials
- Cleartext passwords used by scheduled tasks
- Service credentials
- Sometimes RDP session passwords

---

## 🔑 Why LSA Secrets Matter

Some secrets may be stored in plaintext or be decryptable.

This makes them valuable for credential reuse.

Access generally requires:

```text
SYSTEM / Administrator
        ↓
LSARPC
        ↓
LSA Secrets
```

---

## 🛠️ Tool

The supplied material uses:

```text
secretsdump.py
```

with local administrator credentials.

---

# 4️⃣ DPAPI Vault

**DPAPI** stands for:

```text
Data Protection API
```

It is Windows' built-in cryptographic system for protecting application secrets for users.

Examples include:

- Saved Wi-Fi passwords
- Browser credentials
- RDP credentials
- Other application secrets

---

## 🔐 DPAPI Model

```text
User Password
      ↓
Key protecting DPAPI Master Key
      ↓
DPAPI Master Key
      ↓
Protected Credential Data
```

Master keys are stored under:

```text
%APPDATA%\Microsoft\Protect
```

An attacker who has the required master key and user's logon password may be able to decrypt DPAPI-protected secrets.

---

# 5️⃣ NTDS.dit

`NTDS.dit` is the core Active Directory database.

It exists on:

```text
Domain Controllers
```

It stores information including:

- Domain user accounts
- Computer objects
- Service principals
- NTLM password hashes
- Kerberos key material

---

# 🚨 Why NTDS.dit Is Extremely Valuable

NTDS.dit represents the authentication authority for the domain.

Obtaining its credential material can provide the ability to impersonate domain users and services.

---

## 🛠️ Techniques Mentioned

```text
secretsdump.py -just-dc
```

```text
lsadump::dcsync
```

These can retrieve domain credential material through domain replication mechanisms when the account has the required privileges.

---

# 📊 Credential Store Summary

| Store | What It Holds | Why It Exists | Access Method | Tool / Command |
|---|---|---|---|---|
| LSASS Memory | NTLM hashes, Kerberos tickets, sometimes plaintext | Seamless authentication | Dump `lsass.exe` memory | Mimikatz |
| SAM + SYSTEM | Local account hashes | Local authentication | Export hives + BootKey | `lsadump::sam` |
| LSA Secrets | Cached domain creds, service credentials | Offline logon / stored service credentials | LSARPC / registry secrets | `secretsdump.py` |
| DPAPI Vault | Saved application credentials | User-level secure storage | User token / decrypted master key | `vault::list`, `vault::cred` |
| NTDS.dit | Domain accounts, NTLM hashes, Kerberos keys | Domain authentication | DRSUAPI / offline database | `secretsdump.py -just-dc` |

---

# 🧠 Key Takeaways

- Credential material exists in multiple Windows and AD locations.
- LSASS contains live authentication material.
- SAM contains local account hashes.
- SYSTEM provides key material needed to process the SAM.
- LSA Secrets can contain service and cached credential material.
- DPAPI protects application credentials for individual users.
- NTDS.dit contains the domain's central credential database.
- The required privilege level differs between credential stores.
