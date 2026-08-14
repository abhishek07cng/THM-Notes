# 🛡️ Mitigation

> **Topic:** Introduction to AD Breaching
>
> **Section:** Mitigation

---

# 📖 Overview

After exploring techniques for breaching an AD environment, the focus shifts to defensive controls.

Understanding mitigation is important for both:

- Defenders
- Offensive security practitioners

Knowing what controls should exist helps identify when they are absent.

---

# 🔐 Secrets Management

Credentials in:

- Git repositories
- CI/CD build logs
- Configuration files

can provide easy initial access.

## Recommended Controls

### Use Dedicated Secrets Vaults

Examples:

```text
HashiCorp Vault
Azure Key Vault
AWS Secrets Manager
```

Credentials should be stored and retrieved through dedicated secrets-management systems rather than embedded in source code or configuration files.

---

### Pre-Commit Secret Scanning

Use tools such as:

```text
TruffleHog
Gitleaks
```

to scan for secrets before commits reach version control.

---

### Audit Git History

Regularly audit repositories for historical credential exposure.

Important:

> Removing a secret from the latest commit does not remove it from Git history.

---

### Rotate Exposed Credentials

When a secret is exposed:

```text
Detect Exposure
      ↓
Remove Secret
      ↓
Rotate Credential
      ↓
Invalidate Old Credential
```

Removing the secret from code alone is not enough.

---

### Protect CI/CD Logs

- Mask secrets
- Redact sensitive values
- Restrict access to build output

---

# 🔑 Password Policies and Account Lockout

Password spraying succeeds partly because users choose predictable passwords.

## Recommended Controls

### Password Length

Use a minimum password length of:

```text
14+ characters
```

---

### Ban Common Passwords

Block:

- Common passwords
- Organisation-specific patterns
- Company name + year
- Season + year

The source mentions Azure AD Password Protection as an example of a mechanism for enforcing banned-password policies.

---

### Avoid Organisation-Wide Default Passwords

New accounts should receive:

```text
Unique
+
Random
+
Initial Password
```

---

### Configure Lockout Thresholds Carefully

A threshold that is too low can cause operational disruption.

A threshold that is too high gives attackers more room to spray.

The source describes:

```text
5–10 attempts
+
30-minute observation window
```

as a common balance.

---

### Monitor Distributed Authentication Failures

Password spraying often produces:

```text
Many Accounts
     ↓
Authentication Failures
```

even when no individual account reaches the lockout threshold.

This pattern should trigger an alert.

---

# 🖨️ Device Hardening

LDAP passback relied on a printer with:

- Default credentials
- Plaintext LDAP

## Change Default Credentials

Change administrative credentials on:

- Printers
- Scanners
- MFPs
- IoT devices

before deployment.

---

## Use LDAPS

Prefer:

```text
LDAPS — Port 636
```

instead of:

```text
LDAP — Port 389
```

Encrypted LDAP prevents a passback attack from simply capturing plaintext credentials.

---

## Restrict Device Administration

Restrict administrative interfaces by:

- IP address
- VLAN

Device management interfaces should not be broadly accessible from general user networks.

---

## Use Low-Privilege Service Accounts

LDAP integrations should use dedicated service accounts.

Avoid:

```text
Domain Admin
```

for device integrations.

The service account should have only the read access required for the device's function.

---

## Asset Management

Include network devices in:

- Vulnerability scanning
- Asset inventories
- Security reviews

---

# 📁 File Share Security

File-based coercion was possible because a share allowed overly permissive write access.

## Least Privilege

Users should only have write access where it is genuinely required.

---

## Monitor Suspicious Files

Monitor for files such as:

```text
.url
.lnk
.scf
desktop.ini
```

These files should rarely appear on normal data shares.

---

## Audit Share Activity

Look for suspicious patterns such as:

```text
New File Appears
      ↓
Burst of SMB Authentication Attempts
      ↓
External IP
```

---

# 🔐 NTLM Hardening

The file-based coercion attack captured NTLMv2 because Windows automatically attempted NTLM authentication to the attacker-controlled listener.

---

## Disable NTLMv1

Enforce NTLMv2 as the minimum authentication level.

The source gives the Group Policy setting:

```text
Network Security:
LAN Manager authentication level

Send NTLMv2 response only.
Refuse LM & NTLM
```

---

## Enforce SMB Signing

SMB signing helps prevent relay attacks that intercept and forward NTLM authentication.

---

## Block Outbound SMB

Block unnecessary outbound:

```text
TCP 445
```

at the network perimeter.

The source notes that there is rarely a legitimate reason for internal workstations to initiate SMB connections to external IP addresses.

---

## Plan NTLM Deprecation

Microsoft has begun moving away from NTLM in favour of Kerberos.

Organisations should plan migration where possible.

---

# 🌐 Network Segmentation and Access Control

Many techniques were possible because services were accessible from networks where they did not need to be reachable.

---

## Management VLANs

Segment management interfaces such as:

- Printer administration
- Jenkins dashboards
- Git servers

into dedicated management networks.

---

## Restrict Internal Services

Services should only be accessible to users and systems that require them.

For example:

> A Jenkins instance used by a development team should not necessarily be reachable from the entire corporate network.

---

## MFA

Enable MFA on:

- Internet-facing services
- VPN portals
- Email
- Remote-access gateways
- Critical internal services

MFA reduces the impact of a compromised password because the password alone is no longer sufficient for authentication.

---

# 📊 Attack → Mitigation Mapping

| Attack / Weakness | Recommended Controls |
|---|---|
| Exposed Git credentials | Secrets vaults, pre-commit scanning, Git-history audits, credential rotation |
| CI/CD credential exposure | Mask logs, restrict build output, use secrets management |
| Password spraying | Strong passwords, banned-password lists, careful lockout policies, distributed-failure monitoring |
| LDAP passback | Change default credentials, use LDAPS, restrict device management, low-privilege service accounts |
| File-based coercion | Least-privilege share permissions, suspicious-file monitoring, share auditing |
| NTLM capture | NTLMv1 disablement, NTLMv2 enforcement, SMB signing, outbound SMB restrictions |
| Broad service exposure | Network segmentation and access control |
| Compromised passwords | MFA |

---

# 🧠 Defensive Workflow

```text
Identify Attack Surface
        ↓
Remove Unnecessary Exposure
        ↓
Protect Credentials
        ↓
Harden Authentication
        ↓
Restrict Network Access
        ↓
Monitor Authentication
        ↓
Detect Suspicious Activity
        ↓
Respond and Rotate Credentials
```

---

# 💡 Key Takeaways

- Secrets should be stored in dedicated vaults rather than source code.
- Git history must be audited because deleted secrets can remain recoverable.
- Exposed credentials should be rotated immediately.
- Password policies should prioritise long passwords and banned-password lists.
- Default organisation-wide passwords should be avoided.
- Password spraying should be detected through distributed authentication failures.
- Network devices should have default credentials changed.
- LDAPS should be preferred over plaintext LDAP.
- Device service accounts should have minimal privileges.
- File shares should enforce least privilege.
- Suspicious `.url`, `.lnk`, `.scf` and `desktop.ini` files can be monitored.
- NTLMv1 should be disabled.
- SMB signing helps prevent NTLM relay attacks.
- Unnecessary outbound SMB should be blocked.
- Organisations should plan for NTLM deprecation.
- Management services should be segmented.
- MFA reduces the impact of compromised passwords.
