# 🛡️ Mitigation & Detection

> **Topic:** Intro to AD Lateral Movement
>
> **Focus:** Defensive controls that directly address the lateral movement techniques demonstrated in the room.

---

# 📖 Overview

Every lateral movement technique demonstrated in the room has corresponding defensive controls.

The major defensive themes are:

```text
Prevent Credential Reuse
Restrict Administrative Access
Protect Authentication Material
Restrict Network Paths
Segment Critical Systems
Monitor Authentication and Execution
```

---

# 1️⃣ Windows LAPS

**Local Administrator Password Solution (LAPS)** prevents the same local Administrator password from being reused across multiple machines.

---

# 🔐 Why LAPS Matters

Without LAPS:

```text
WRK Local Admin Password
        ↓
Same Password
        ↓
SERVER1 Local Admin
        ↓
Repeated NT Hash
        ↓
Pass-the-Hash
```

With unique passwords:

```text
WRK Admin Password ≠ SERVER1 Admin Password
```

A compromised local Administrator hash becomes much less useful against other machines.

---

# 🆕 Windows LAPS

The supplied material highlights Windows LAPS as the modern built-in solution.

It provides capabilities including:

- Unique local Administrator passwords.
- Automatic password rotation.
- Secure storage in Active Directory.
- Password encryption support.
- Password history.
- Entra ID storage.
- Domain Controller DSRM password management.

---

# 2️⃣ Restrict Local Administrator Rights

Remote execution methods such as:

```text
PsExec
WMI
DCOM
SMBExec
```

generally require administrative access on the target.

Therefore:

> Remove unnecessary local Administrator privileges.

---

## Least Privilege

Standard users should not be local Administrators on unrelated workstations.

Administrative access should use dedicated admin accounts separate from daily-use accounts.

---

# 🏛️ Tiered Administration

The supplied material describes:

```text
Tier 0
→ Domain Controllers / Domain Administration

Tier 1
→ Servers

Tier 2
→ Workstations
```

A Tier 0 account should not routinely log into lower-trust systems.

This reduces the possibility that a Domain Admin credential becomes exposed on a workstation.

---

# 3️⃣ SMB Signing

Several lateral movement techniques rely on SMB.

SMB signing helps verify the integrity and origin of SMB communication.

The supplied material identifies these Group Policy settings:

```text
Microsoft network server:
Digitally sign communications (always)

Microsoft network client:
Digitally sign communications (always)
```

Both should be enabled in environments where the policy is applicable.

---

# 4️⃣ Restrict NTLM

Pass-the-Hash relies on NTLM.

Therefore, restricting NTLM reduces the PtH attack surface.

The supplied material highlights:

```text
Restrict NTLM: NTLM authentication in this domain
```

and:

```text
Restrict NTLM: Audit NTLM authentication in this domain
```

---

# 🔄 Recommended Approach

```text
Audit NTLM Usage
       ↓
Identify Dependencies
       ↓
Remediate Legacy Dependencies
       ↓
Gradually Restrict NTLM
```

Fully disabling NTLM can be difficult when legacy applications depend on it.

---

# 5️⃣ Credential Guard

Credential Guard uses virtualization-based security (VBS) to isolate sensitive LSASS credential material.

Conceptually:

```text
Credential Material
       ↓
Protected Environment
       ↓
Harder to Extract from LSASS
```

The supplied material notes that Credential Guard can prevent tools such as Mimikatz from extracting NTLM hashes and Kerberos tickets from memory, even with SYSTEM access.

---

# 6️⃣ Host Firewall Rules

Workstations generally should not need unrestricted workstation-to-workstation access over:

```text
SMB 445
WinRM 5985
RDP 3389
```

Windows Defender Firewall rules can restrict inbound administrative protocols between workstations while allowing legitimate management systems.

---

# 7️⃣ Network Segmentation

Separate:

```text
Workstations
Servers
Domain Controllers
```

into appropriately controlled network segments.

---

## Recommended Segmentation Principles

- Separate workstations and servers using VLANs.
- Restrict east-west workstation traffic.
- Place Domain Controllers in dedicated hardened subnets.
- Restrict inbound DC access to authorised management systems.
- Block unnecessary outbound Internet access from Domain Controllers.

---

# 🖥️ 8️⃣ Privileged Access Workstations

A **Privileged Access Workstation (PAW)** is a hardened machine used specifically for privileged administration.

The key principle is:

```text
Daily Workstation
       ≠
Tier 0 Administration Workstation
```

---

# 🔐 Why PAWs Matter

If an administrator's normal workstation is compromised:

```text
Compromised Workstation
        ↓
No Tier 0 Credentials Present
        ↓
No DA Hash to Harvest
        ↓
No DA TGT to Steal
        ↓
Reduced Domain Compromise Risk
```

---

# 9️⃣ Monitoring & Detection

Preventive controls are important, but monitoring is also required.

---

# 📊 Important Windows Events

| Event ID | Log | What to Monitor |
|---|---|---|
| `4624 Type 3` | Security | Network logon |
| `4624 Type 10` | Security | RDP / remote interactive logon |
| `4648` | Security | Explicit credential usage |
| `7045` | System | New service installed |
| `4698` | Security | Scheduled task created |
| `4688` | Security | Process creation |

---

# 🚨 PsExec Detection

A strong PsExec indicator is:

```text
Event ID 7045
+
Short/randomised service name
```

The supplied material notes that legitimate services rarely use short random names.

---

# 🔍 Sysmon

The supplied material identifies Sysmon as an additional source of process-level telemetry.

Particularly useful events include:

```text
Sysmon Event ID 1
→ Process creation

Sysmon Event ID 10
→ Process access
```

Event ID 10 can be valuable for identifying unexpected processes opening handles to:

```text
lsass.exe
```

---

# 📈 Behavioural Detection

Lateral movement often creates unusual authentication patterns.

Examples:

```text
One account authenticating to many hosts rapidly
```

```text
Workstation → Workstation SMB
```

```text
Service account → Interactive logon
```

These patterns can be strong signals when combined with appropriate logging.

---

# 🧠 Defensive Mapping

| Attack Technique | Defensive Control |
|---|---|
| Local Admin hash reuse | Windows LAPS |
| Excessive local admin rights | Least privilege |
| PsExec / SMBExec | SMB controls + firewall + monitoring |
| Pass-the-Hash | Restrict NTLM + Credential Guard |
| DA credential exposure | Tiered administration + PAWs |
| Pivoting | Network segmentation |
| Remote execution | Host firewall rules |
| Suspicious service creation | Event 7045 monitoring |
| Suspicious process access | Sysmon |
| RDP abuse | Event 4624 Type 10 monitoring |

---

# 💡 Key Takeaways

- Unique local Administrator passwords significantly reduce hash-reuse attacks.
- Least privilege reduces the number of hosts where stolen credentials can be used.
- Tiered administration limits where privileged accounts can authenticate.
- SMB signing strengthens SMB security.
- NTLM restriction reduces the Pass-the-Hash attack surface.
- Credential Guard protects sensitive LSASS material.
- Host firewalls reduce unnecessary lateral connectivity.
- Network segmentation limits attacker movement and pivoting.
- PAWs prevent Tier 0 credentials from being exposed on normal workstations.
- Event monitoring can reveal remote execution and abnormal authentication patterns.
