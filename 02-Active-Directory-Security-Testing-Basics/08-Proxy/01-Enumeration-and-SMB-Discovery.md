# 01 — Enumeration & SMB Discovery

## 1. Port Scanning

The target was scanned with:

```bash
nmap -p- 10.49.157.149
```

### Key Findings

The host exposed several Active Directory-related services:

| Port | Service | Significance |
|---:|---|---|
| 53 | DNS | Domain name resolution |
| 88 | Kerberos | AD authentication |
| 135 | MSRPC | Windows RPC |
| 139 | NetBIOS | SMB/NetBIOS |
| 389 | LDAP | Active Directory LDAP |
| 445 | SMB | File sharing / Windows networking |
| 464 | Kerberos password change | AD/Kerberos |
| 593 | RPC over HTTP | Windows RPC |
| 636 | LDAPS | Secure LDAP |
| 3268 | Global Catalog LDAP | AD Global Catalog |
| 3269 | Global Catalog LDAPS | Secure Global Catalog |
| 3389 | RDP | Remote Desktop |
| 9389 | .NET Message Framing | AD Web Services-related service |
| 49668+ | RPC | Dynamic Windows RPC |

Nmap also identified:

```text
Host: DC01
Domain: ctf.local
NetBIOS Domain: CTF
DNS Name: DC01.ctf.local
```

SMB signing was reported as enabled and required.

## 2. Anonymous RPC Enumeration

I tested anonymous RPC access:

```bash
rpcclient -U "" 10.49.157.149 -N
```

Then attempted:

```text
enumdomusers
enumdomgroups
querydominfo
lookupnames Domain
```

Every attempt returned:

```text
NT_STATUS_ACCESS_DENIED
```

### Lesson

Anonymous RPC enumeration was not available, so I moved on to other exposed services.

## 3. Local Recon / Interesting Files

The AttackBox contained:

```text
IT-Credentials-Backup.txt
IT-Onboarding-Checklist.txt
IT-Portal.html
```

The archived credential file contained:

```text
helpdesk.bob : Welcome123!
it.admin     : ITAdmin2019!
```

However, both accounts were explicitly marked disabled.

The onboarding checklist was more interesting because it revealed service accounts:

```text
File Scanner (svc.scanner)
    Runs every 2 minutes.
    Enumerates IT-Shared for new files to process.

Database Backup (svc.mssql)
    Handles nightly MSSQL backups.
    Member of Backup Operators.
```

This made `svc.scanner` an important candidate for further investigation.

## 4. IT-Shared SMB Share

Anonymous access to the share was successful:

```bash
smbclient //10.49.157.149/IT-Shared -N
```

Listing the share:

```text
IT-Credentials-Backup.txt
IT-Onboarding-Checklist.txt
IT-Portal.html
```

Importantly, the share was writable.

I tested this with:

```text
put test.bat
```

The file was successfully uploaded.

### Important Observation

This was the turning point:

```text
Anonymous SMB access
        +
Writable IT-Shared
        +
Automated svc.scanner
        ↓
Potential NTLM credential capture
```

The onboarding file explicitly said `svc.scanner` periodically enumerates `IT-Shared`, making the writable share particularly interesting.

