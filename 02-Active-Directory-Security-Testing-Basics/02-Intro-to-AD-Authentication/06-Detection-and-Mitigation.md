````
# 🛡️ Detection & Mitigation

> **TryHackMe Room:** Intro to AD Authentication
>
> **Task:** 6 — Detection & Mitigation

---

# 📖 Overview

Windows logs a security event for authentication attempts.

Understanding these **Windows Security Event IDs** is important for detecting attacks against Active Directory authentication.

This task maps important Event IDs to the authentication attacks covered in the previous tasks and provides practical mitigation techniques.

```text
Authentication Attempt
        ↓
Windows Security Event
        ↓
Event ID
        ↓
Detection
        ↓
Investigation
        ↓
Mitigation
````

---

# 📊 Key Windows Event IDs

| Event ID | Log | Description |
|---|---|---|
| **4624**               | Security | Successful logon. Check the Authentication Package and Logon Type.                                 |
| **4625**               | Security | Failed logon. Useful for detecting password spraying.                                              |
| **4768**               | Security | Kerberos Ticket Granting Ticket (TGT) requested.                                                   |
| **4769**               | Security | Kerberos service ticket requested. Important for Kerberoasting detection.                          |
| **4771**               | Security | Kerberos pre-authentication failed. Useful for detecting AS-REP Roasting and brute-force activity. |

---

# 🔍 Event ID 4624 — Successful Logon

Event ID:

```
4624
```

is generated when a successful logon occurs.

When investigating authentication attacks, important fields include:

-  Authentication Package 
-  Logon Type 
-  Source Network Address 

These fields can help determine how authentication occurred.

---

# 🔴 Detecting NTLM-Based Attacks

When NTLM authentication is used, several fields in Event ID 4624 are particularly important.

---

## Authentication Package

The:

```
Authentication Package
```

field indicates which authentication protocol was used.

For NTLM authentication:

```
Authentication Package: NTLM
```

Kerberos logons will show:

```
Authentication Package: Kerberos
```

---

## Logon Type

A:

```
Logon Type: 3
```

indicates a **network logon**.

This is commonly associated with network-based authentication such as:

-  SMB 
-  WinRM 

A network logon using NTLM against a high-value target can be a strong indicator of a **Pass-the-Hash** attempt.

---

## Source Network Address

The source network address is also useful during investigation.

When the Domain Controller validates an NTLM logon, the resulting Event ID 4624 can lack the source IP address.

This can make attribution more difficult.

By comparison, the equivalent Kerberos events:

```
4768
4769
```

populate the client address field.

---

# 🧠 NTLM Detection Example

A potentially suspicious authentication event might contain:

```
Event ID: 4624
Authentication Package: NTLM
Logon Type: 3
Target: Domain Controller
```

This combination deserves investigation, particularly when NTLM is being used against a high-value system.

```
NTLM
  +
Network Logon
  +
High-Value Target
       ↓
Potential Pass-the-Hash Activity
```

---

# 🎫 Detecting Kerberoasting

Kerberoasting can be detected by monitoring:

```
Event ID 4769
```

Event ID 4769 is generated whenever a **Kerberos service ticket** is requested.

A Kerberoasting attack can produce a spike of 4769 events within a short period.

---

# 🔎 Indicators of Kerberoasting

## 1. High Volume of 4769 Events

Look for:

```
Many 4769 Events
       ↓
Short Time Window
       ↓
Same User Account
       ↓
Multiple Service Accounts
```

A high volume of service-ticket requests from a single account in a short period can be suspicious.

---

## 2. Ticket Encryption Type

Another important field is:

```
Ticket Encryption Type
```

The supplied material highlights:

```
0x17
```

which represents:

```
RC4-HMAC
```

Modern environments generally use stronger AES encryption such as:

```
0x12
```

for AES-256.

Therefore, an RC4 ticket request from an account that supports AES may indicate a deliberate downgrade intended to make offline cracking easier.

---

# 🔥 Detecting AS-REP Roasting

Monitor:

```
Event ID 4771
```

Event ID 4771 indicates:

```
Kerberos pre-authentication failed
```

A spike in 4771 events across many accounts within a short period can indicate:

-  AS-REP Roasting activity 
-  Brute-force attempts 
-  Attempts to identify accounts with Kerberos pre-authentication disabled 

---

# 📊 Authentication Attack Detection Summary

| Attack | Important Event ID | What to Look For |
|---|---:|---|
| **Pass-the-Hash**                        | 4624 | NTLM + Logon Type 3 + high-value target       |
| **Password Spraying**                    | 4625 | Repeated failures across multiple accounts    |
| **Kerberoasting**                        | 4769 | Large number of service-ticket requests       |
| **AS-REP Roasting**                      | 4771 | Spike of Kerberos pre-authentication failures |
| **Kerberos Authentication**              | 4768 | TGT requests                                  |
| **Kerberos Service Access**              | 4769 | Service-ticket requests                       |

---

# 🛡️ Mitigations

Detection is only one part of Active Directory security.

The next step is reducing the attack surface and protecting authentication material.

---

# 1️⃣ Pass-the-Hash

## Attack

An attacker uses a stolen NTLM hash to authenticate without knowing the plaintext password.

## Mitigation

The supplied material recommends:

-  Add privileged accounts to the **Protected Users** group. 
-  Disable NTLM where Kerberos is available. 

```
Privileged Accounts
        ↓
Protected Users
        +
Reduce NTLM Usage
        ↓
Reduced PtH Risk
```

---

# 2️⃣ NTLM Relay

## Attack

An attacker intercepts NTLM authentication and relays it to another service.

## Mitigation

Recommended controls include:

-  Enforce **SMB signing** 
-  Enable **Extended Protection for Authentication (EPA)** on: 
  -  LDAP 
  -  AD CS 

```
NTLM Relay
    ↓
SMB Signing
EPA
    ↓
Reduced Relay Risk
```

---

# 3️⃣ Kerberoasting

## Attack

An attacker requests service tickets for accounts with registered SPNs and attempts to crack the tickets offline.

## Mitigation

Use:

-  Strong service-account passwords 
-  Random service-account passwords 
-  Group Managed Service Accounts (**gMSA**) 

```
Service Accounts
       ↓
Strong Random Passwords
       OR
      gMSA
       ↓
Reduced Kerberoasting Risk
```

---

# 4️⃣ Golden Ticket

## Attack

An attacker who compromises the KRBTGT account's password hash can forge Kerberos tickets.

## Mitigation

Protect the:

```
KRBTGT
```

account carefully.

After a suspected KRBTGT compromise, reset its password:

```
Twice
```

The double reset is important because the previous KRBTGT password is also cached.

```
KRBTGT Compromise
       ↓
Reset Password
       ↓
Reset Password Again
       ↓
Invalidate Forged Tickets
```

---

# 5️⃣ Password Spraying

## Attack

An attacker attempts a commonly used password against many accounts.

Example:

```
Common Password
      ↓
User01
User02
User03
User04
User05
```

## Mitigation

Recommended controls include:

-  Configure account lockout policies. 
-  Monitor Event ID **4625**. 
-  Look for repeated authentication failures across multiple accounts. 

```
4625 Events
    ↓
Multiple Accounts
    ↓
Repeated Failures
    ↓
Investigate Password Spraying
```

---

# 📋 Attack vs Mitigation

| Attack | Mitigation |
|---|---|
| **Pass-the-Hash**     | Add privileged accounts to Protected Users; disable NTLM where Kerberos is available |
| **NTLM Relay**        | Enforce SMB signing; enable EPA on LDAP and AD CS                                    |
| **Kerberoasting**     | Use strong random service-account passwords or migrate to gMSA                       |
| **Golden Ticket**     | Protect KRBTGT; reset its password twice after suspected compromise                  |
| **Password Spraying** | Configure account lockout policies; monitor Event ID 4625                            |

---

# 🧠 Important Event IDs to Remember

For Active Directory authentication security, remember these five Event IDs:

```
4624 → Successful Logon
4625 → Failed Logon
4768 → Kerberos TGT Request
4769 → Kerberos Service Ticket Request
4771 → Kerberos Pre-Authentication Failed
```

A useful revision mnemonic:

```
4624 → Success
4625 → Failure

4768 → TGT
4769 → Service Ticket
4771 → Pre-Auth Failure
```

---

# 🔄 Detection Workflow

A basic defensive workflow can be represented as:

```
Authentication Event
        ↓
Identify Event ID
        ↓
Check Authentication Protocol
        ↓
Check User
        ↓
Check Source
        ↓
Check Frequency
        ↓
Identify Suspicious Pattern
        ↓
Investigate
        ↓
Apply Mitigation
```

---

# 🎯 Attack Detection Mapping

```
                 AD Authentication
                        │
          ┌─────────────┴─────────────┐
          │                           │
         NTLM                      Kerberos
          │                           │
     ┌────┴────┐              ┌───────┴────────┐
     │         │              │                │
     ▼         ▼              ▼                ▼
   PtH      NTLM Relay    Kerberoasting   AS-REP Roasting
     │         │              │                │
     ▼         ▼              ▼                ▼
   4624      4624            4769             4771
```

---

# 🛡️ Defensive Priorities

The major defensive priorities from this task are:

### 1. Reduce NTLM Usage

Prefer Kerberos where possible.

### 2. Protect Privileged Accounts

Use security controls such as:

```
Protected Users
```

for privileged accounts.

### 3. Protect Service Accounts

Use:

-  Strong random passwords 
-  gMSA 

### 4. Protect KRBTGT

A compromised KRBTGT account can lead to Golden Ticket attacks.

### 5. Monitor Authentication Events

Pay particular attention to:

```
4624
4625
4768
4769
4771
```

### 6. Harden Network Authentication

Use:

-  SMB signing 
-  Extended Protection for Authentication (EPA) 

where appropriate.

---

# 💡 Key Takeaways

-  Windows records authentication-related security events. 
-  Event ID **4624** represents a successful logon. 
-  Event ID **4625** represents a failed logon. 
-  Event ID **4768** represents a Kerberos TGT request. 
-  Event ID **4769** represents a Kerberos service-ticket request. 
-  Event ID **4771** represents a Kerberos pre-authentication failure. 
-  NTLM authentication can be identified through the Authentication Package field. 
-  Logon Type 3 represents a network logon. 
-  NTLM network logons against high-value targets can indicate Pass-the-Hash activity. 
-  Kerberoasting can produce a spike in Event ID 4769. 
-  RC4-HMAC (`0x17`) ticket requests can be suspicious when AES is supported. 
-  Event ID 4771 can help identify AS-REP Roasting or brute-force activity. 
-  Protected Users can help reduce Pass-the-Hash risk for privileged accounts. 
-  SMB signing helps protect against NTLM relay attacks. 
-  EPA can help protect LDAP and AD CS against relay attacks. 
-  Strong random service-account passwords reduce Kerberoasting risk. 
-  gMSAs can improve service-account password management. 
-  The KRBTGT account must be carefully protected. 
-  After a suspected KRBTGT compromise, the password should be reset twice. 
-  Event ID 4625 can help detect password spraying. 

---

# 📚 Quick Revision Table

| Event ID | Meaning | Security Relevance |
|---|---|---|
| **4624**                          | Successful Logon          | Investigate authentication method and logon type |
| **4625**                          | Failed Logon              | Password spraying / brute-force detection        |
| **4768**                          | TGT Requested             | Kerberos authentication monitoring               |
| **4769**                          | Service Ticket Requested  | Kerberoasting detection                          |
| **4771**                          | Pre-Authentication Failed | AS-REP Roasting / brute-force detection          |

---

# 📚 References

-  TryHackMe — Intro to AD Authentication 

````
