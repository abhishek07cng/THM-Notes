# 🔄 What Is Lateral Movement?

> **Topic:** Intro to AD Lateral Movement
>
> **Focus:** Understanding lateral movement, its requirements, major techniques, and the tools used throughout the room.

---

# 📖 Overview

In an Active Directory environment, **lateral movement** is the process of moving from one compromised host to another using valid authentication material.

This can include:

- Plaintext credentials
- NTLM hashes
- Kerberos tickets

Unlike initial access, where an attacker may exploit a vulnerability or spray passwords, lateral movement relies on legitimate authentication mechanisms.

> **Core idea:** Move → Harvest → Move Again.

---

# 🎯 Why Does Lateral Movement Matter?

A compromised workstation is usually only a foothold.

More valuable targets may exist elsewhere in the environment:

- Domain Controllers
- Service accounts
- Backup servers
- Servers containing sensitive information
- Other privileged systems

The attacker can use credentials harvested from one host to authenticate to another.

```text
Compromised Host
      ↓
Harvest Credentials
      ↓
Authenticate to Another Host
      ↓
Compromise New Host
      ↓
Harvest More Credentials
      ↓
Move Again
```

This iterative cycle continues until the objective is reached.

---

# 🏛️ Three Pillars of Lateral Movement

The supplied material divides lateral movement into three broad categories:

```text
1. Remote Execution
2. Credential Reuse
3. Pivoting
```

---

# 1️⃣ Remote Execution

Remote execution means running commands on another host through legitimate Windows administration protocols.

Examples include:

- PsExec
- WinRM
- WMI
- DCOM
- SMBExec
- AtExec

These mechanisms exist for legitimate administration, which is also why they can be useful during authorised security testing.

Detailed practical execution is covered in the next task.

---

# 2️⃣ Credential Reuse

Credential reuse means using authentication material obtained from one system to authenticate to another.

Examples:

```text
Plaintext Password
NTLM Hash
Kerberos Ticket
```

One important technique is:

```text
Pass-the-Hash
```

NTLM authentication uses a challenge-response mechanism based on the NT hash, allowing an attacker who possesses the underlying NT hash to authenticate without knowing the plaintext password.

Other techniques include:

```text
Pass-the-Ticket
Overpass-the-Hash
```

---

# 3️⃣ Pivoting

Networks are often segmented.

For example:

```text
Attacker Network
      X
      |
      X
Internal Network
      |
Domain Controller
```

A compromised host that has access to both networks can be used as a relay.

```text
AttackBox
    ↓
Compromised Host
    ↓
Internal Network
    ↓
Target
```

Common approaches include:

- SSH local port forwarding
- SSH dynamic port forwarding
- SOCKS proxies

---

# 🔑 Requirements for Lateral Movement

The supplied material identifies two important requirements.

## 1. Valid Authentication Material

You generally need one of:

```text
Plaintext Password
NTLM Hash
Kerberos Ticket
```

for an account that can authenticate to the target.

---

## 2. Appropriate Access on the Target

Most remote execution techniques require local Administrator privileges.

For example, PsExec needs access to:

```text
ADMIN$
Service Control Manager
```

WinRM is more flexible and can also allow members of:

```text
Remote Management Users
```

---

# ⚠️ Local Admin ≠ Domain Admin

Having local Administrator access on a workstation does not automatically make an account a Domain Administrator.

However:

```text
Local Administrator
        +
Domain Admin credential cached on host
        ↓
Potential path to domain compromise
```

This is why privileged accounts should not authenticate to lower-trust systems unnecessarily.

---

# 🧰 Toolbox

| Tool | Purpose |
|---|---|
| `psexec.py` | SMB-based remote execution |
| `wmiexec.py` | WMI-based remote execution |
| `smbexec.py` | SMB/service-based execution |
| `dcomexec.py` | DCOM-based execution |
| `atexec.py` | Task Scheduler-based execution |
| `nxc` / NetExec | Multi-protocol authentication and execution |
| Evil-WinRM | Interactive WinRM PowerShell sessions |
| SSH | Tunnelling and pivoting |
| Mimikatz | Credential/ticket operations |
| Rubeus | Kerberos operations |
| Chisel | Tunnelling |
| Ligolo-ng | Advanced pivoting |

---

# 💡 Key Takeaways

- Lateral movement means moving between compromised hosts using valid authentication material.
- The three major categories are remote execution, credential reuse and pivoting.
- Plaintext passwords, NTLM hashes and Kerberos tickets can all enable movement.
- Most remote execution techniques require administrative access on the target.
- WinRM can also allow Remote Management Users.
- Local Administrator does not automatically mean Domain Administrator.
- A compromised host can become a stepping stone to more valuable systems.
