# 📂 SMB Enumeration

> **Topic:** AD Basic Enumeration
>
> **Section:** Network Enumeration with SMB

---

# 📖 Overview

**Server Message Block (SMB)** is widely used in Windows environments for:

- File sharing
- Remote administration
- Inter-system communication

SMB enumeration can reveal accessible shares, permissions and potentially sensitive files.

---

# 🔌 Important SMB-Related Ports

| Port | Purpose |
|---:|---|
| `139` | NetBIOS Session Service / legacy SMB |
| `445` | Modern SMB |

Other AD-related services may also be present on the Domain Controller:

| Port | Service |
|---:|---|
| `88` | Kerberos |
| `135` | MS-RPC |
| `389` | LDAP |
| `636` | LDAPS |

---

# 🔎 Discovering SMB Services

Nmap can be used to identify Windows services:

```bash
nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP
```

Example AD-related results:

```text
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
636/tcp  open  tcpwrapped
```

---

# 📁 Listing SMB Shares with smbclient

When valid credentials are unavailable, test for anonymous SMB access.

```bash
smbclient -L //10.211.11.10 -N
```

### Options

```text
-L
```

Lists available shares.

```text
-N
```

Attempts the connection without requesting a password.

---

# 📄 Example Output

```text
Anonymous login successful

Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
AnonShare       Disk
C$              Disk      Default share
IPC$            IPC       Remote IPC
NETLOGON        Disk      Logon server share
SharedFiles     Disk
SYSVOL          Disk      Logon server share
UserBackups     Disk
```

Potentially interesting non-standard shares may include:

```text
AnonShare
SharedFiles
UserBackups
```

---

# 🗺️ SMB Enumeration with smbmap

`smbmap` is designed to enumerate SMB shares and display permissions.

Example:

```bash
./smbmap.py -H 10.211.11.10
```

Example permissions:

```text
Disk                     Permissions
----                     -----------
ADMIN$                   NO ACCESS
AnonShare                READ, WRITE
C$                       NO ACCESS
IPC$                     NO ACCESS
NETLOGON                 NO ACCESS
SharedFiles              READ, WRITE
SYSVOL                   NO ACCESS
UserBackups              READ, WRITE
```

This quickly highlights shares that may be accessible.

---

# 🔍 Nmap SMB Share Enumeration

Nmap also provides an SMB share enumeration script:

```bash
nmap -p445 --script smb-enum-shares 10.211.11.10
```

This can help identify:

- Share names
- Read access
- Write access
- Access restrictions

---

# 🔓 Accessing an Anonymous Share

Connect to a share:

```bash
smbclient //10.211.11.10/SharedFiles -N
```

Then list its contents:

```text
smb: \> ls
```

Example:

```text
Mouse_and_Malware.txt
```

Download the file:

```text
smb: \> get Mouse_and_Malware.txt
```

Exit:

```text
smb: \> exit
```

---

# 🔐 Authenticated SMB Access

If credentials are available, `smbclient` can use:

```text
--user=USERNAME
--password=PASSWORD
```

or:

```text
-U 'username%password'
```

For domain accounts, specify the domain using:

```text
-W DOMAIN
```

---

# 🧰 Other SMB Enumeration Tools

## Impacket smbclient

A Python-based SMB client included in the Impacket toolkit.

The source identifies the AttackBox location as:

```text
/opt/impacket/examples/
```

---

## CrackMapExec

CrackMapExec can perform:

- SMB enumeration
- Credential testing
- Network service enumeration
- Post-exploitation activities

---

## enum4linux / enum4linux-ng

Can perform extensive SMB-based enumeration.

Example:

```bash
enum4linux -a TARGET_IP
```

Redirecting the output to a file can make large results easier to review.

---

# 📂 What Can SMB Shares Contain?

Potentially useful content includes:

```text
Configuration files
Backup files
Scripts
Documents
Usernames
Passwords
Service credentials
```

Writable shares deserve particular attention because users may upload additional files to them.

---

# ⚠️ Security Risk

Anonymous SMB shares are risky because they may expose internal data without authentication.

Legacy systems may still require anonymous access, but this can create an unintended information-disclosure path.

---

# 💡 Key Takeaways

- SMB is a major source of AD enumeration information.
- `smbclient` can list and access shares.
- `smbmap` quickly shows share permissions.
- Nmap can enumerate SMB shares with `smb-enum-shares`.
- Anonymous access should always be checked within authorized scope.
- Readable shares may contain sensitive files.
- Writable shares can introduce additional risks.
- Useful SMB enumeration alternatives include Impacket, CrackMapExec and enum4linux-ng.
