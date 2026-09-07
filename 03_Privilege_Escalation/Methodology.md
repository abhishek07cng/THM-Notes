# Privilege Escalation Methodology

## 1. Initial Access

Start from the privileges you actually obtained. Do not assume the compromised account is useless or already privileged.

## 2. Situational Awareness

### Linux
- `whoami`
- `id`
- `hostname`
- `uname -a`
- `cat /proc/version`
- `cat /etc/issue`
- `cat /etc/os-release`

### Windows
- `whoami`
- `whoami /priv`
- `whoami /groups`
- `systeminfo`

## 3. Category-Based Enumeration

### Users and Groups
Find local users, group membership, administrative memberships, service accounts, and potentially exposed credentials.

### File and Directory Permissions
Look for SUID/SGID binaries, writable files/directories, sensitive files, Windows ACL weaknesses, and writable program/service locations.

### Services
Identify privileged services, their executable paths, service accounts, configuration files, and permissions.

### Scheduled Tasks / Cron
Find jobs running with elevated privileges and inspect the scripts/binaries they execute.

### Credential Storage
Check shell history, environment variables, configuration files, Windows credential stores, deployment files, registry entries, and application configuration.

### Network Configuration
Review interfaces, routes, listening services, and possible pivoting opportunities.

## 4. Prioritise Findings

Prefer direct, reliable escalation paths. Stored administrator/root credentials may be more direct than a scheduled job that runs later. Also look for **chains** where two individually weak findings combine into a working escalation path.

## 5. Manual Before Automated

Automated tools save time but can miss vectors. Use tools such as LinPEAS, WinPEAS, PowerUp, LinEnum, Linux Smart Enumeration, and pspy as supplements to manual enumeration.

## 6. Vulnerability Research

If configuration-based paths are exhausted, research vulnerable software and kernel versions. Confirm version, architecture, distribution, exploit requirements, and operational impact before using a public exploit.

## 7. Exploit and Verify

After an escalation attempt, verify with: `whoami`, `id`, or the appropriate Windows identity command. Confirm access to a resource that was previously unavailable.

## Decision Tree

```text
Low-privileged shell
      ↓
Identify user + OS + privileges
      ↓
Users / Files / Services / Tasks / Credentials / Network
      ↓
Is there a direct configuration weakness? ── Yes → Exploit → Verify
      │
      No
      ↓
Check installed software + kernel versions
      ↓
Known CVE / public exploit? ── Yes → Evaluate → Exploit → Verify
      │
      No
      ↓
Automated enumeration + pspy / deeper manual checks
```
