# 🔐 NetNTLM Authentication

> **TryHackMe Room:** Intro to AD Authentication
>
> **Task:** 3 — NetNTLM Authentication

---

# 📖 Overview

**NetNTLM**, often simply called **NTLM**, is a challenge-response authentication protocol that has existed since the early days of Windows NT.

Although Kerberos has largely replaced NTLM as the default authentication protocol in modern Windows environments, NTLM is still widely used.

Common scenarios include:

- Legacy systems
- Workgroup environments
- Fallback authentication when Kerberos is unavailable

---

# 🧩 NTLM Versions

NTLM has several versions.

## NTLMv1

The original version of NTLM.

It is now considered:

> **Highly insecure**

---

## NTLMv2

An improved version of NTLM with stronger cryptography.

However, NTLMv2 is still vulnerable to various attacks.

---

# 🔄 How NTLM Authentication Works

A key characteristic of NTLM is that the client authenticates to the **service it wants to access**, and that service then verifies the user's identity against the **Domain Controller**.

The authentication process follows several steps.

---

## Step 1 — Client Requests Access

The client sends a request to access a service and provides their username.

```text
Client
   ↓
Service
   │
   └── Username
```

---

## Step 2 — Server Generates Challenge

The server generates a random **16-byte number** called a:

> **Challenge / Nonce**

The server sends this challenge to the client.

```text
Server
   ↓
Random 16-byte Challenge
   ↓
Client
```

---

## Step 3 — Client Creates Response

The client encrypts the challenge using the **NT hash** of the user's password.

The resulting response is sent back to the server.

```text
NT Hash
   +
Challenge
   ↓
Encrypted Response
   ↓
Server
```

---

## Step 4 — Server Forwards Authentication

The server forwards the following information to the Domain Controller:

- Username
- Original challenge
- Client's response

```text
Client
   ↓
Username + Response
   ↓
Server
   ↓
Domain Controller
```

---

## Step 5 — Domain Controller Verifies Response

The Domain Controller retrieves the user's NT hash from its database.

It then uses the same hash to encrypt the original challenge.

```text
Stored NT Hash
      +
Original Challenge
      ↓
Expected Response
```

The Domain Controller compares this result with the response received from the client.

---

## Step 6 — Authentication Result

If both responses match:

```text
Client Response
       =
DC Calculated Response
       ↓
Authentication Successful
```

If they do not match:

```text
Client Response
       ≠
DC Calculated Response
       ↓
Authentication Failed
```

The server then receives the result from the Domain Controller and either grants or denies access.

---

# 🔐 Password Is Not Sent

An important characteristic of NTLM is that the user's actual password is **never sent over the network**.

Instead, only the encrypted response to the challenge is transmitted.

The user proves knowledge of the password without directly revealing the password itself.

---

# 📊 NTLM Authentication Flow

```text
Client
   │
   │  Username
   ▼
Server
   │
   │  Random 16-byte Challenge
   ▼
Client
   │
   │  Challenge encrypted using NT Hash
   ▼
Server
   │
   │  Username + Challenge + Response
   ▼
Domain Controller
   │
   │  Retrieve NT Hash
   │
   │  Calculate Expected Response
   ▼
Verification
   │
   ├── Match ──────► Authentication Success
   │
   └── No Match ───► Authentication Failure
```

---

# ✅ Benefits of NTLM

Despite its age, NTLM has several advantages that explain why it remains in use.

---

## 1. Simplicity

NTLM is relatively simple to implement.

It does not require additional infrastructure such as a:

> **Key Distribution Center (KDC)**

---

## 2. No Time Synchronisation Required

Unlike Kerberos, NTLM does not depend on synchronized clocks between systems.

This can make NTLM easier to deploy in certain environments.

---

## 3. Fallback Compatibility

NTLM can serve as a fallback when Kerberos authentication fails.

This ensures that users can still authenticate in situations where Kerberos cannot be used.

---

## 4. Workgroup Support

NTLM can be used in **non-domain environments**, such as Windows workgroups, where Kerberos is not available.

---

# ❌ Drawbacks of NTLM

Although NTLM provides compatibility and simplicity, it has significant security weaknesses.

---

## 1. No Mutual Authentication

The client cannot verify the identity of the server.

This leaves NTLM vulnerable to:

- Man-in-the-middle attacks
- Authentication relay attacks

---

## 2. Weak Cryptography

NTLMv1 uses:

- DES encryption
- Unsalted hashes

These weaknesses make NTLMv1 vulnerable to rapid password cracking with modern hardware.

NTLMv2 provides stronger cryptography but still uses unsalted hashes in memory.

---

## 3. Vulnerable to Relay Attacks

An attacker can potentially intercept NTLM authentication and relay it to another service.

This can allow unauthorized access without requiring the attacker to crack the credentials.

---

## 4. Pass-the-Hash Attacks

The NT hash is directly involved in the NTLM authentication process.

Therefore, an attacker who obtains a user's NT hash can authenticate without knowing the user's actual password.

This attack is known as:

> **Pass-the-Hash (PtH)**

---

## 5. Slower Performance

Each authentication request requires communication with the Domain Controller.

In large environments, this can introduce additional authentication overhead.

---

# 📍 When Is NTLM Used?

Even in modern Active Directory environments where Kerberos is the default, NTLM may still be used in several situations.

---

## 1. Domain Controller Cannot Be Reached

NTLM can be used when the client cannot reach a Domain Controller to obtain a Kerberos ticket.

---

## 2. Accessing a Resource by IP Address

Kerberos requires a **Service Principal Name (SPN)**, which relies on DNS names.

Therefore, accessing a resource directly by IP address can result in NTLM being used instead.

Example:

```text
\\192.168.11.51\SHARE1
```

instead of:

```text
\\SERVER1.thm.loc\SHARE1
```

---

## 3. Target Service Has No SPN

If the target service does not have a registered **Service Principal Name (SPN)** in Active Directory, NTLM may be used instead.

---

## 4. Non-Domain-Joined Systems

NTLM can authenticate to systems that are not joined to the Active Directory domain.

---

## 5. Legacy Applications

Older applications may still require NTLM authentication.

---

# 🧪 Practical Demonstration — NTLM Authentication

> **TryHackMe Lab**

The room demonstrates NTLM authentication using **Impacket**.

Impacket is a collection of Python scripts that implement various network protocols, including protocols commonly used in Active Directory environments.

The TryHackMe AttackBox provides the Impacket examples in:

```text
/opt/impacket/examples/
```

---

# 💻 TryHackMe Example

From the AttackBox, the room uses `smbclient.py` to authenticate to a target SMB share using NTLM credentials.

```bash
smbclient.py thm.loc/claire:'Password123!'@192.168.11.51
```

Example output:

```text
Impacket v0.10.1.dev1+20230316.112532.f0ac44bd - Copyright 2022 Fortra

Type help for list of commands
#
```

The command and example values above are preserved from the TryHackMe lab. :contentReference[oaicite:1]{index=1}

---

# 📂 Accessing the SMB Share

Once connected, the room demonstrates listing the available shares using:

```text
shares
```

Then connect to `SHARE1` using:

```text
use SHARE1
```

After accessing the share, the flag can be retrieved as instructed by the TryHackMe room. :contentReference[oaicite:2]{index=2}

---

# 🔍 Command Breakdown

```bash
smbclient.py thm.loc/claire:'Password123!'@192.168.11.51
```

### `smbclient.py`

An Impacket tool used to interact with SMB services.

---

### `thm.loc`

The Active Directory domain used in the TryHackMe lab.

---

### `claire`

The username used in the TryHackMe demonstration.

---

### `'Password123!'`

The password used in the TryHackMe demonstration.

---

### `192.168.11.51`

The target machine's IP address used in the lab.

---

# ⚙️ Behind the Scenes

The TryHackMe room describes the following sequence:

```text
Your client sent your username to the target.
              ↓
The target responded with a challenge.
              ↓
Your client encrypted the challenge
using your password's NT hash.
              ↓
The target forwarded your credentials
to the Domain Controller.
              ↓
The Domain Controller verified the response.
              ↓
Access was granted.
```

:contentReference[oaicite:3]{index=3}

---

# 🧠 Complete NTLM Demonstration Flow

```text
AttackBox
    │
    │ smbclient.py
    │ Username + Password
    ▼
Target SMB Server
    │
    │ NTLM Challenge
    ▼
AttackBox
    │
    │ NT Hash → Challenge Response
    ▼
Target SMB Server
    │
    │ Authentication Request
    ▼
Domain Controller
    │
    │ Verify Response
    ▼
Authentication Result
    │
    ▼
Target SMB Server
    │
    ▼
SMB Share Access
```

---

# 🎯 Why This Matters for Security Testing

Understanding the NTLM authentication flow is important because several Active Directory attacks target weaknesses in this process.

Important attacks related to NTLM include:

- Pass-the-Hash
- NTLM Relay
- NTLM downgrade attacks
- Offline password cracking

These attacks are covered later in the Active Directory learning path.

---

# 💡 Key Takeaways

- NTLM is a challenge-response authentication protocol.
- NTLMv1 is considered highly insecure.
- NTLMv2 provides stronger cryptography but remains vulnerable to attacks.
- The client authenticates to the service it wants to access.
- The service verifies the authentication with the Domain Controller.
- A random 16-byte challenge is generated by the server.
- The client uses the NT hash of the password to create the response.
- The plaintext password is never sent over the network.
- NTLM does not provide mutual authentication.
- NTLM is vulnerable to relay attacks.
- NTLM enables Pass-the-Hash attacks when an attacker obtains the NT hash.
- NTLM may still be used when Kerberos is unavailable or cannot be used.
- NTLM is also common with legacy applications and non-domain systems.
- Impacket's `smbclient.py` can be used to demonstrate NTLM authentication.

---

# 📚 References

- TryHackMe — Intro to AD Authentication