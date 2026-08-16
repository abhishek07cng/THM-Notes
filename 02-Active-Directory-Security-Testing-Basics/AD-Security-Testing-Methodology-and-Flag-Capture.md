# 🏴‍☠️ AD Security Testing Methodology & Flag-Capture Playbook

> **Purpose:** A practical decision-making guide for solving authorised Active Directory labs/CTFs using the techniques covered in this repository.
>
> This is a **methodology/reference file**, not a single CTF solution.

---

# 🎯 The Golden Rule

Do not start by randomly running tools.

Use:

```text
ENUMERATE
   ↓
UNDERSTAND
   ↓
HYPOTHESISE
   ↓
TEST
   ↓
VALIDATE
   ↓
ENUMERATE AGAIN
   ↓
MOVE / ESCALATE
   ↓
OBJECTIVE
```

Every successful action should create a reason for the next action.

---

# 1. 🗺️ Start With Network Enumeration

First determine:

```text
What hosts exist?
What ports are open?
What services are exposed?
Which host is the Domain Controller?
What is the domain name?
```

Start with:

```bash
nmap -p- TARGET
```

Then perform service/version enumeration where appropriate:

```bash
nmap -sC -sV TARGET
```

---

# 2. 🏛️ Identify the Domain Controller

Look for AD indicators:

```text
53   DNS
88   Kerberos
389  LDAP
445  SMB
464  Kerberos password service
636  LDAPS
3268 Global Catalog
3269 Global Catalog LDAPS
```

Use DNS:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.DOMAIN DC_IP
```

and:

```bash
nslookup -type=SRV _kerberos._tcp.DOMAIN DC_IP
```

Record:

```text
Domain:
DC:
DC IP:
NetBIOS Name:
```

---

# 3. 🔓 Test Unauthenticated Enumeration

Before obtaining credentials, check what is exposed anonymously.

## SMB

```bash
smbclient -L //TARGET -N
```

If a share is accessible:

```bash
smbclient //TARGET/SHARE -N
```

Check:

```text
dir
get FILE
put TEST_FILE
```

A **writable** anonymous share deserves special attention.

---

## RPC

```bash
rpcclient -U "" TARGET -N
```

Try relevant enumeration:

```text
enumdomusers
enumdomgroups
querydominfo
```

If you receive:

```text
NT_STATUS_ACCESS_DENIED
```

move on.

---

# 4. 👤 Build a Username List

Potential sources:

```text
OSINT
Corporate naming conventions
GitHub/GitLab
Public documentation
Job postings
Internal documents
CTF-provided files
```

Common formats:

```text
first.last
firstlast
flast
first
last.first
```

Save candidates:

```text
usernames.txt
```

---

# 5. 🔎 Validate Usernames

Use Kerbrute in authorised labs:

```bash
kerbrute userenum -d DOMAIN --dc DC_IP usernames.txt
```

Save results:

```bash
kerbrute userenum -d DOMAIN --dc DC_IP usernames.txt -o valid_users.txt
```

Think:

```text
Unknown Users
     ↓
Valid Users
     ↓
Credential Attack Surface
```

---

# 6. 🔐 Check Password Policy Before Spraying

Before password spraying, determine what the lab exposes about:

```text
Minimum Password Length
Lockout Threshold
Lockout Duration
Password Complexity
```

The purpose is to avoid blindly spraying and causing unnecessary account lockouts.

---

# 7. 🧪 Look for Credential Exposure

Search the available lab material and exposed services for:

```text
Passwords
API Keys
Tokens
Connection Strings
Service Credentials
Configuration Files
Backup Files
CI/CD Secrets
```

Useful sources:

```text
Git history
Jenkins
Internal portals
SMB shares
Configuration files
```

For Git:

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

For a lab repository:

```bash
trufflehog git file:///path/to/repo
```

---

# 8. 🔑 Once You Have Credentials — Change Your Strategy

Do **not** continue treating the target as unauthenticated.

Record:

```text
Username:
Password / Hash:
Domain:
Target:
```

Then move to:

```text
Authenticated Enumeration
```

---

# 9. 👥 Enumerate the Domain

Look for:

```text
Users
Groups
Computers
Domain Admins
Enterprise Admins
Service Accounts
SPNs
Delegation
Sessions
ACLs
```

Useful tooling:

```text
NetExec
PowerShell ActiveDirectory
PowerView
BloodHound
SharpHound
```

---

# 10. 🩸 BloodHound Workflow

Think of BloodHound as a **relationship/attack-path map**.

Look for paths such as:

```text
Your User
   ↓
Group Membership
   ↓
Local Admin
   ↓
Computer
   ↓
Session
   ↓
Privileged User
   ↓
Domain Admin
```

Also inspect:

```text
GenericAll
GenericWrite
WriteDACL
WriteOwner
AddMember
ForceChangePassword
AllowedToDelegate
AdminTo
HasSession
```

Do not just look for "Domain Admin."

Look for:

> **What relationship connects my current identity to a more privileged identity?**

---

# 11. 🎫 Check Kerberos Attack Opportunities

After authenticated enumeration, inspect:

```text
SPNs
Pre-authentication configuration
Delegation
Kerberos tickets
```

Potential paths covered in this module:

```text
Kerberoasting
AS-REP Roasting
Pass-the-Ticket
Overpass-the-Hash
Delegation Abuse
Golden Ticket
Silver Ticket
```

---

# 12. 🔥 AS-REP Roasting Decision

Ask:

```text
Does a user have Kerberos pre-authentication disabled?
```

If yes, the account may be vulnerable to AS-REP Roasting.

Conceptually:

```text
User Without Pre-Auth
       ↓
AS-REP
       ↓
Offline Crackable Material
       ↓
Password Recovery
```

---

# 13. 🍖 Kerberoasting Decision

Ask:

```text
Which domain accounts have SPNs?
```

Service accounts with weak passwords can be interesting.

Conceptually:

```text
Valid Domain User
       ↓
SPN
       ↓
TGS Request
       ↓
Offline Cracking
       ↓
Service Account Password
```

Relevant tool:

```bash
GetUserSPNs.py
```

---

# 14. 🔐 Credential Store Enumeration

If you gain suitable privileges on a Windows host, ask:

```text
What credentials could exist here?
```

Check conceptually:

```text
LSASS
SAM + SYSTEM
LSA Cache / Secrets
DPAPI
NTDS.dit
```

Tools covered:

```text
Mimikatz
secretsdump.py
reg.exe
```

---

# 15. 🧠 Understand the Credential Type

Before attempting reuse, identify what you actually obtained.

## NT Hash

```text
32 hex characters
```

Potentially relevant to:

```text
Pass-the-Hash
```

## NetNTLMv2

Challenge-response capture.

Typical path:

```text
Capture
 ↓
Crack OR Relay
```

It is **not** the same as an NT hash.

## Kerberos Ticket

Typical path:

```text
CCache / .kirbi
 ↓
Ticket Injection / Kerberos-Aware Tool
```

---

# 16. 🔄 Lateral Movement Decision

Once you have valid credentials, ask:

```text
Which hosts can this identity access?
```

Use:

```bash
nxc smb TARGET -u USER -p 'PASSWORD'
```

or, where authorised:

```bash
nxc smb TARGET -u USER -H NT_HASH
```

If administrative access is indicated:

```text
(Pwn3d!)
```

then consider the appropriate remote execution technique from the lab.

---

# 17. 🖥️ Remote Execution

Common techniques in the notes:

```text
PsExec
WMIExec
SMBExec
DCOMExec
AtExec
Evil-WinRM
```

Selection depends on:

```text
Available Protocol
Required Privileges
Network Reachability
Detection / Noise
```

---

# 18. 🔁 The Critical Rule: Enumerate Again

After moving to a new host:

> **Start enumeration again.**

Do not assume the new host is only useful for execution.

Ask:

```text
whoami
hostname
ipconfig
```

Then inspect the environment according to the lab scope:

```text
Users
Groups
Sessions
Shares
Configuration
Credentials
Interesting Files
Network Connectivity
```

The workflow is:

```text
Host A
 ↓
Credential
 ↓
Host B
 ↓
New Information
 ↓
New Credential
 ↓
Host C
```

---

# 19. 🕳️ Check for Pivoting

If the target is not directly reachable:

```text
Can the compromised host reach it?
```

If yes, consider:

```text
SSH -L
SSH -D
ProxyChains
Chisel
Ligolo-ng
```

Conceptually:

```text
AttackBox
    ↓
Pivot Host
    ↓
Internal Network
    ↓
Target
```

---

# 20. 🏁 How to Capture the Flag

When you believe you have reached the objective host:

## Step 1 — Confirm Identity

```cmd
whoami
```

## Step 2 — Confirm Host

```cmd
hostname
```

## Step 3 — Determine Privilege

```cmd
whoami /all
```

## Step 4 — Locate the Objective

Use the **CTF-provided objective clues** and inspect the relevant directories/files.

Common lab locations may include:

```text
Desktop
Documents
C:\Users\<user>\
C:\Users\Administrator\
C:\Users\Public\
```

Do not assume the flag is always in one location.

---

# 21. 🏴 Flag-Capture Validation

When you find the flag:

```text
1. Confirm you are on the correct host.
2. Confirm your current identity.
3. Read the flag.
4. Record the exact path.
5. Record how you reached that host.
6. Record the credential / privilege transition.
```

Your final notes should look like:

```text
Initial Access
      ↓
Credential
      ↓
Enumeration
      ↓
Attack Path
      ↓
Privilege Escalation
      ↓
Lateral Movement
      ↓
Pivot
      ↓
Objective Host
      ↓
Flag
```

---

# 22. 🧪 Example: Proxy CTF Methodology

The Proxy CTF demonstrates the methodology especially well.

## Enumeration

```text
Nmap
 ↓
DC01
 ↓
SMB
```

## Discovery

```text
IT-Shared
 ↓
Writable
```

## Context

```text
Onboarding Checklist
 ↓
svc.scanner
 ↓
Runs every 2 minutes
 ↓
Scans IT-Shared
```

## Credential Capture

```text
Trigger
 ↓
Responder
 ↓
svc.scanner NetNTLMv2
```

## Credential Recovery

```text
Hashcat
 ↓
svc.scanner Password
```

## Privilege Path

```text
Delegation Enumeration
 ↓
Constrained Delegation
 ↓
getST.py
 ↓
Administrator Ticket
```

## Final Access

```text
Kerberos CCache
 ↓
smbexec.py
 ↓
DC01
```

The lesson is:

> **The attack path came from connecting multiple enumeration results.**

---

# 23. 🧩 Decision Tree

```text
START
  │
  ▼
Can I reach the target?
  │
  ├── NO → Identify network/pivot path
  │
  └── YES
       │
       ▼
Are AD services exposed?
       │
       ├── NO → General enumeration
       │
       └── YES
            │
            ▼
Can I enumerate anonymously?
            │
       ┌────┴────┐
       YES       NO
        │         │
        ▼         ▼
     SMB/RPC   Find credentials
     LDAP/DNS      │
        │          ▼
        └────→ Authenticated
               Enumeration
                    │
                    ▼
             Find attack paths
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
     SPNs       Delegation     Sessions
       │            │            │
       ▼            ▼            ▼
 Kerberoast    Kerberos       Token /
               Abuse          Credential
       │            │            │
       └────────────┼────────────┘
                    ▼
             Lateral Movement
                    │
                    ▼
             New Host Found
                    │
                    ▼
          ENUMERATE AGAIN
                    │
                    ▼
              Objective
                    │
                    ▼
                 FLAG
```

---

# 24. 📝 What to Record During Every CTF

Create a small working table:

| Item | Value |
|---|---|
| Target IP | |
| Domain | |
| Domain Controller | |
| Open Ports | |
| Valid Users | |
| Credentials | |
| Hashes | |
| Interesting Shares | |
| Service Accounts | |
| SPNs | |
| Delegation | |
| Privileged Groups | |
| Compromised Hosts | |
| Pivot Host | |
| Objective Host | |
| Flag Location | |

This prevents losing important information during a long CTF.

---

# 25. 🧠 After Capturing the Flag

Do not immediately close the lab.

Write a post-exploitation summary:

```text
Initial Access:
Why it worked:

Credential Obtained:
How:

Enumeration:
What was discovered:

Privilege Escalation:
Why it worked:

Lateral Movement:
How:

Pivot:
Why it was required:

Final Access:
How:

Flag:
Where:

Detection:
What could have detected it?

Mitigation:
How could it have been prevented?
```

---

# 🎯 Final AD CTF Checklist

```text
[ ] Identify target IP
[ ] Full port scan
[ ] Identify AD services
[ ] Identify Domain Controller
[ ] Identify domain name
[ ] Test anonymous SMB
[ ] Test anonymous RPC
[ ] Enumerate DNS
[ ] Build username list
[ ] Validate usernames
[ ] Check password policy
[ ] Search exposed credentials
[ ] Obtain valid credentials
[ ] Validate credentials
[ ] Enumerate users/groups/computers
[ ] Enumerate SPNs
[ ] Check AS-REP Roasting
[ ] Check Kerberoasting
[ ] Check delegation
[ ] Run BloodHound/SharpHound where appropriate
[ ] Identify privileged paths
[ ] Check credential stores when privileged
[ ] Test authorised lateral movement
[ ] Enumerate every newly compromised host
[ ] Identify pivot requirements
[ ] Pivot if required
[ ] Reach objective host
[ ] Confirm identity
[ ] Confirm hostname
[ ] Locate flag
[ ] Record exact flag path
[ ] Document complete attack chain
[ ] Document detection and mitigation
```

---

# 🏆 The Skill to Build

Do not aim to become someone who knows:

```text
100 commands
```

Aim to become someone who can look at:

```text
One username
One host
One share
One SPN
One group relationship
One delegation setting
One credential
```

and ask:

> **"What does this allow me to investigate next?"**

That is the core Active Directory security-testing skill this repository is designed to develop.
