# ⚙️ Service Accounts, Environment Variables and Registry

> **Topic:** AD Authenticated Enumeration
>
> **Focus:** Identifying service accounts, service configurations, installed software and potentially sensitive registry data.

---

# 📖 Overview

After understanding the current user, groups, sessions and system, continue by examining:

```text
Services
   ↓
Service Accounts
   ↓
Environment Variables
   ↓
Registry
   ↓
Installed Applications
   ↓
Potentially Sensitive Configuration
```

---

# 👤 Service Accounts

Not every Windows account represents a human user.

**Service accounts** are local or domain accounts used by applications and services.

They may have elevated privileges because the service needs them to perform its function.

The supplied material highlights that service accounts often have static passwords that may not expire.

---

# 🔎 Enumerating Services with WMIC

The supplied material uses WMIC to identify the account associated with each Windows service.

```cmd
wmic service get Name,StartName
```

> **Requirement:** Administrator privileges.

---

# 📊 Understanding StartName

The `StartName` field identifies the account under which the service runs.

Common service identities include:

```text
LocalSystem
NT AUTHORITY\LocalService
NT AUTHORITY\NetworkService
NT SERVICE\SomeServiceName
```

A particularly interesting result is a domain account:

```text
DomainName\username
```

Such an account may deserve further investigation because it could potentially be reused elsewhere or have a weaker-than-expected password.

---

# 🐚 PowerShell Equivalent

```powershell
Get-WmiObject Win32_Service | select Name, StartName
```

Example:

```text
Name              StartName
----              ---------
AJRouter          NT AUTHORITY\LocalService
ALG               NT AUTHORITY\LocalService
AmazonSSMAgent    LocalSystem
Appinfo           LocalSystem
[...]
```

---

# 🛠️ Service Control Manager — SC

`sc` communicates with the Windows Service Control Manager.

List all services:

```cmd
sc query state= all
```

> **Requirement:** Administrator privileges.

---

# 🔍 Filtering Services

Because the output can be large:

```cmd
sc query state= all | find "DHCP"
```

Example:

```text
SERVICE_NAME: DHCP
```

---

# ⚙️ Query Service Configuration

Once the service name is known:

```cmd
sc qc DHCP
```

Example:

```text
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: DHCP
START_TYPE         : 2   AUTO_START
BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p
DISPLAY_NAME       : DHCP Client
SERVICE_START_NAME : NT Authority\LocalService
```

---

# 🔑 SERVICE_START_NAME

The important field is:

```text
SERVICE_START_NAME
```

It identifies the account that starts the service.

---

# 🌐 Environment Variables

Environment variables may reveal information about:

- Installed software
- Development tools
- User directories
- Domain information
- Application paths

Example:

```cmd
set
```

PowerShell:

```powershell
Get-ChildItem Env:
```

or:

```powershell
dir env:
```

---

# 🗃️ Windows Registry

The Registry contains persistent configuration information.

It is extremely large, so targeted enumeration is more practical than manually inspecting every key.

---

# 🔐 Saved Auto-Logon Credentials

The supplied material identifies this registry location:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

Potentially interesting values include:

```text
DefaultUsername
DefaultPassword
AutoAdminLogon
```

Query an individual value:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUsername
```

Example:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    DefaultUsername    REG_SZ    Strategos
```

If an auto-logon password has been stored insecurely, the supplied material notes that it may be available in plaintext.

---

# 🔒 Security Cache

Another registry location mentioned is:

```text
HKLM\Security\Cache
```

The supplied material notes that:

- Administrator access is required.
- Credentials are hashed.
- The hashes require cracking.

---

# 📦 Installed Applications

Installed applications can be enumerated through:

```cmd
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

This can reveal software installed on the machine.

Example:

```text
AddressBook
Connection Manager
DirectDrawEx
DXM_Runtime
Fontcore
IE40
IE4Data
```

Installed software can help identify:

- Server roles
- Development tools
- Security software
- Potentially vulnerable applications
- Software with known/default credentials

---

# 🔎 Searching the Registry

Search for a specific keyword:

```cmd
reg query HKLM /f "password" /t REG_SZ /s
```

### Breakdown

```text
HKLM
```

Searches the HKEY_LOCAL_MACHINE hive.

```text
/f "password"
```

Searches for the specified string.

```text
/t REG_SZ
```

Limits the search to string values.

```text
/s
```

Searches recursively.

---

# 💡 Key Takeaways

- Service accounts may have higher privileges than normal users.
- `wmic service get Name,StartName` identifies service accounts.
- `sc query state= all` lists services.
- `sc qc <ServiceName>` reveals service configuration.
- Environment variables can expose installed software and configuration hints.
- The Winlogon registry key may contain auto-logon configuration.
- Installed applications can be enumerated from the Uninstall registry path.
- Targeted registry searches can reveal potentially sensitive configuration.
