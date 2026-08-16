# 🏴‍☠️ Active Directory Security Testing — Complete Notes

> **Purpose:** Complete revision repository for Active Directory security testing, built from the AD learning material and CTF/THM notes developed throughout this module.
>
> **Scope:** AD fundamentals → authentication → breaching → enumeration → credential harvesting → lateral movement → detection/mitigation → practical CTF methodology.

---

# 📖 Overview

This repository documents the complete Active Directory security-testing workflow covered in this learning module.

The progression is:

```text
AD Fundamentals
      ↓
Authentication
      ↓
Initial Breach
      ↓
Basic Enumeration
      ↓
Authenticated Enumeration
      ↓
Credential Harvesting
      ↓
Lateral Movement
      ↓
Privilege Escalation / Attack Path
      ↓
Domain Controller Access
      ↓
Detection & Mitigation
```

The goal is not to memorize individual commands.

The goal is to understand:

> **What should I enumerate next, why am I enumerating it, what does the result mean, and what attack path can it create?**

---

# 🗂️ AD Module Structure

| Module | Focus |
|---|---|
| `02-Intro-to-AD-Authentication` | NTLM, Kerberos, authentication material and authentication weaknesses |
| `03-Introduction-to-AD-Breaching` | Initial access, OSINT, username enumeration, credential discovery, spraying and coercion |
| `04-AD-Basic-Enumeration` | Network, SMB, LDAP, RPC, Kerberos and password-policy enumeration |
| `05-AD-Authenticated-Enumeration` | AS-REP Roasting, Windows enumeration, service accounts, BloodHound, PowerShell and PowerView |
| `06-Credential-Harvesting` | LSASS, SAM, LSA Secrets, DPAPI, NTDS.dit and credential extraction |
| `07-Intro-to-AD-Lateral-Movement` | Remote execution, Pass-the-Hash, Pass-the-Ticket and pivoting |
| `06-Detection-and-Mitigation.md` | Authentication detection and defensive controls |
| `Proxy-CTF-Writeup` | Practical AD CTF application and complete attack-chain documentation |

> Module numbering reflects the structure developed during the notes project. The separate detection/mitigation material is retained because it is a cross-cutting defensive reference.

---

# 🎯 Learning Objectives

By completing this repository, you should be able to:

- Explain how Active Directory is structured.
- Explain NTLM and Kerberos authentication.
- Identify Domain Controllers and AD-related services.
- Perform unauthenticated AD reconnaissance in an authorised lab.
- Enumerate SMB, LDAP, RPC and Kerberos.
- Discover valid usernames.
- Understand password spraying and its risks.
- Identify exposed credentials and service accounts.
- Perform authenticated AD enumeration.
- Understand AS-REP Roasting and Kerberoasting.
- Understand BloodHound attack-path analysis.
- Identify credential stores on Windows systems.
- Understand LSASS, SAM, LSA Secrets, DPAPI and NTDS.dit.
- Understand remote execution methods.
- Understand Pass-the-Hash and Kerberos ticket abuse.
- Understand pivoting through a compromised host.
- Recognise common AD attack chains.
- Map attacks to detection events and mitigations.

---

# 🧠 The Core AD Mental Model

When approaching an AD lab, think in layers:

```text
1. What is the network?
        ↓
2. Where is the Domain Controller?
        ↓
3. What services are exposed?
        ↓
4. Can I enumerate without credentials?
        ↓
5. Can I obtain valid credentials?
        ↓
6. What can those credentials access?
        ↓
7. What users / groups / computers exist?
        ↓
8. What privileged relationships exist?
        ↓
9. Where are credentials stored?
        ↓
10. Can I move to another host?
        ↓
11. Can I reach the DC?
        ↓
12. What is the final authorised objective?
```

---

# 🔐 Authentication Foundation

The two major authentication protocols covered are:

```text
NTLM
Kerberos
```

## NTLM

```text
Client
   ↓
Challenge
   ↓
Response using NT hash
   ↓
Verification
   ↓
Access
```

Important concepts:

- NT hash
- NetNTLMv2
- Pass-the-Hash
- NTLM Relay

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
```

Important concepts:

- TGT
- TGS
- SPN
- KRBTGT
- Kerberoasting
- AS-REP Roasting
- Pass-the-Ticket
- Overpass-the-Hash
- Golden Ticket
- Silver Ticket

---

# 🔎 Initial AD Breach

When no credentials are available, the first objective is to discover an entry point.

Typical workflow:

```text
OSINT
 ↓
Username Discovery
 ↓
DNS / AD Enumeration
 ↓
Credential Discovery
 ↓
Password Spraying / Coercion
 ↓
Valid AD Credential
```

Potential credential sources covered in the notes include:

```text
Git repositories
Jenkins
Internal portals
Configuration files
Service accounts
Public information
```

---

# 🌐 Basic Enumeration

Once a target is identified:

```text
Host Discovery
 ↓
Port Scan
 ↓
Identify DC
 ↓
SMB
 ↓
LDAP
 ↓
RPC
 ↓
Kerberos
 ↓
Password Policy
 ↓
User Enumeration
```

The objective is to build an **AD map**, not just collect open ports.

---

# 👤 Authenticated Enumeration

Once valid credentials are obtained:

```text
Valid Credentials
       ↓
Domain Users
       ↓
Groups
       ↓
Computers
       ↓
SPNs
       ↓
Delegation
       ↓
Sessions
       ↓
ACLs / Relationships
       ↓
Attack Paths
```

Important tooling includes:

```text
BloodHound
SharpHound
PowerView
PowerShell ActiveDirectory
Impacket
NetExec
```

---

# 🔑 Credential Harvesting

Important Windows credential stores:

```text
LSASS
SAM + SYSTEM
LSA Secrets / Cache
DPAPI
NTDS.dit
```

Conceptually:

```text
Compromised Host
      ↓
Credential Store
      ↓
Credential Material
      ↓
Crack / Reuse / Ticket Operation
      ↓
Higher Privilege
```

---

# 🔄 Lateral Movement

The three major categories covered are:

```text
Remote Execution
Credential Reuse
Pivoting
```

Typical progression:

```text
Compromised Workstation
       ↓
Harvest Credential
       ↓
Authenticate to Server
       ↓
Harvest Again
       ↓
Move Again
```

The key lesson is:

> **Move → Harvest → Move Again.**

---

# 🕳️ Pivoting

If the target network cannot be reached directly:

```text
AttackBox
    X
    |
Compromised Host
    |
Internal Network
    |
Domain Controller
```

Common approaches covered:

```text
SSH -L
SSH -D
ProxyChains
Chisel
Ligolo-ng
```

---

# 🏁 End Goal / Flag-Capture Mindset

A CTF objective should be treated as the **final validation step**, not the only goal.

The workflow should be:

```text
Enumerate
   ↓
Identify Attack Path
   ↓
Gain Access
   ↓
Enumerate Again
   ↓
Escalate / Move
   ↓
Reach Objective Host
   ↓
Locate Objective
   ↓
Read / Capture Flag
   ↓
Document Exact Path
```

Do not stop after obtaining a shell.

Ask:

```text
Who am I?
Where am I?
What privileges do I have?
What files / shares / sessions are available?
What users have authenticated here?
What can this host reach?
Where is the objective?
```

---

# 🧪 Practical CTF Example

The Proxy CTF documented in this repository demonstrates the methodology:

```text
Nmap
 ↓
SMB Enumeration
 ↓
Writable IT-Shared
 ↓
svc.scanner Discovery
 ↓
Responder
 ↓
NetNTLMv2
 ↓
Hashcat
 ↓
svc.scanner
 ↓
Delegation Enumeration
 ↓
Kerberos Service Ticket
 ↓
Administrator Impersonation
 ↓
smbexec.py
 ↓
DC01
```

This is a good example of why enumeration should be **hypothesis-driven**.

The writable share alone was not the final solution.

The important relationship was:

```text
Writable Share
+
Automated Service Account
+
SMB Authentication
```

---

# 🛡️ Detection & Mitigation

Important authentication events:

```text
4624 → Successful Logon
4625 → Failed Logon
4768 → Kerberos TGT Request
4769 → Kerberos Service Ticket Request
4771 → Kerberos Pre-Authentication Failure
```

Major controls covered:

```text
Windows LAPS
Least Privilege
Protected Users
Credential Guard
SMB Signing
EPA
Strong Service Account Passwords
gMSA
Tiered Administration
PAWs
Network Segmentation
Host Firewalls
Authentication Monitoring
```

---

# 📚 Recommended Study Order

```text
01. AD Fundamentals
        ↓
02. Authentication
        ↓
03. AD Breaching
        ↓
04. Basic Enumeration
        ↓
05. Authenticated Enumeration
        ↓
06. Credential Harvesting
        ↓
07. Lateral Movement
        ↓
08. CTF Practice
        ↓
09. Detection & Mitigation
```

---

# 🧠 Final Revision Model

```text
             ACTIVE DIRECTORY
                    │
        ┌───────────┴───────────┐
        │                       │
   Authentication          Enumeration
        │                       │
  ┌─────┴─────┐          ┌──────┴──────┐
  NTLM     Kerberos      SMB LDAP RPC DNS
        │                       │
        └──────────┬────────────┘
                   ↓
            Initial Access
                   ↓
         Credential Discovery
                   ↓
        Authenticated Enumeration
                   ↓
           Attack Path Mapping
                   ↓
          Credential Harvesting
                   ↓
           Lateral Movement
                   ↓
              Pivoting
                   ↓
           Domain Controller
                   ↓
              Objective
```

---

# ⚠️ Lab Safety

Commands, credentials, hashes, IP addresses, domain names and exploitation techniques in these notes are retained as **training/CTF examples**.

Use them only against:

- TryHackMe labs
- Your own lab environments
- Systems where you have explicit authorisation

---

# 📌 Repository Philosophy

These notes are intentionally more than a list of commands.

For every technique, try to remember:

```text
What?
Why?
Prerequisites?
How?
What does the output mean?
What can I do next?
How can it be detected?
How can it be mitigated?
```

That is the skill this repository is designed to build.
