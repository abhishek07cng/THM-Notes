# 🔄 Intro to AD Lateral Movement

> **Purpose:** Structured notes for understanding and practising lateral movement techniques in Active Directory environments.

---

# 📖 Overview

This module covers:

```text
Remote Execution
Credential Reuse
Pivoting
Mitigation & Detection
```

The room demonstrates how a compromised host can become a stepping stone to additional hosts, credentials and ultimately the Domain Controller.

---

# 🗂️ Module Structure

| File | Topic |
|---|---|
| `01-What-is-Lateral-Movement.md` | Definition, requirements and three pillars |
| `02-Remote-Execution-Methods.md` | PsExec, Evil-WinRM and other execution methods |
| `03-Pass-the-Hash-and-Credential-Reuse.md` | PtH, PtT, Overpass-the-Hash and token impersonation |
| `04-Pivoting.md` | SSH forwarding, SOCKS, ProxyChains, Chisel and Ligolo-ng |
| `05-Mitigation-and-Detection.md` | LAPS, least privilege, segmentation and monitoring |
| `06-Complete-Lateral-Movement-Chain.md` | Complete attack chain and revision |
| `Tools.md` | Consolidated tools and commands |

---

# 🧭 Three Pillars

```text
┌─────────────────────┐
│   Remote Execution  │
└─────────────────────┘
          +
┌─────────────────────┐
│   Credential Reuse  │
└─────────────────────┘
          +
┌─────────────────────┐
│      Pivoting       │
└─────────────────────┘
```

---

# 🔗 Complete Chain

```text
Compromised Host
      ↓
Remote Execution
      ↓
Credential Harvesting
      ↓
Pass-the-Hash
      ↓
New Host
      ↓
Higher-Privilege Credentials
      ↓
Pivot
      ↓
Domain Controller
```

---

# 🛠️ Main Tools

```text
Impacket
NetExec
Evil-WinRM
SSH
ProxyChains
xfreerdp
Chisel
Ligolo-ng
Mimikatz
Rubeus
Incognito
Sysmon
```

See [`Tools.md`](Tools.md).

---

# 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain lateral movement in AD.
- Identify the three major lateral movement categories.
- Understand the requirements for remote execution.
- Explain how PsExec works.
- Understand Evil-WinRM and its privilege requirements.
- Distinguish NT hashes from Net-NTLMv2.
- Perform authorised Pass-the-Hash exercises in a lab.
- Understand Pass-the-Ticket and Overpass-the-Hash.
- Understand token impersonation.
- Explain SSH local and dynamic forwarding.
- Configure a SOCKS pivot with ProxyChains.
- Understand Chisel and Ligolo-ng.
- Identify important Windows event IDs for lateral movement.
- Explain LAPS, Credential Guard, PAWs and network segmentation as mitigations.

---

# 🧠 Core Revision

```text
Lateral Movement
→ Move between hosts using valid authentication material.

Remote Execution
→ Execute commands through Windows administration protocols.

Credential Reuse
→ Reuse passwords, hashes or tickets.

Pivoting
→ Route traffic through a compromised host.

Mitigation
→ Reduce privileges, protect credentials, restrict protocols,
  segment networks and monitor authentication.
```

---

# 📚 Source

Based on the supplied **Intro to AD Lateral Movement** training content.
