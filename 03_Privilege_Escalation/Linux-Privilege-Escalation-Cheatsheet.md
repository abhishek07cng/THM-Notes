# Linux Privilege Escalation — Practical Cheat Sheet

> **Purpose:** Fast reference for THM/CTF/authorized Linux labs.
> **Core flow:** Enumerate → Identify leads → Validate → Exploit → Verify.
> This cheat sheet is derived from the Linux Enumeration, Basics, and Automation material previously provided.

---

# 0. FIRST: Get Context

```bash
whoami
id
hostname
uname -a
uname -r
cat /etc/os-release
```

### Ask immediately

```text
Who am I?
What groups am I in?
What OS/kernel/version?
What privileges do I have?
```

---

# 1. USER ENUMERATION

```bash
id
whoami
env
history
sudo -l
cat /etc/passwd
```

### High-value findings

| Finding | Investigate |
|---|---|
| `sudo -l` gives commands | Can they be abused for elevated execution? |
| Interesting environment variables | Paths, credentials, configuration |
| History contains secrets | Passwords/commands |
| Other users | Credentials, readable files, escalation path |
| Password/hash material | Can it be reused in the lab? |

---

# 2. PROCESS ENUMERATION

```bash
ps aux
ps axjf
```

Look for:

- Root processes
- Interesting services
- Scripts
- Unusual commands
- Applications running with elevated privileges

### If something is interesting

```text
Who owns it?
What executable/script is being run?
Can I modify it?
Can I influence its inputs?
```

---

# 3. NETWORK ENUMERATION

```bash
ifconfig
ip addr
netstat -antup
netstat -ano
ss -tulpn
```

Look for:

- Listening services
- Local-only services
- Unexpected ports
- Services running as root
- Applications worth researching

---

# 4. FILE ENUMERATION

## Current directory

```bash
ls -la
```

## Search common interesting files

```bash
find / -name "*.txt" 2>/dev/null
find / -name "*.conf" 2>/dev/null
find / -name "*.config" 2>/dev/null
find / -name "*.bak" 2>/dev/null
```

## Writable files

```bash
find / -writable -type f 2>/dev/null
```

## Writable directories

```bash
find / -writable -type d 2>/dev/null
```

## SUID files

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Ask

```text
Is this file writable?
Is it executed by root?
Is it SUID?
Can it be abused?
```

---

# 5. SUDO — CHECK THIS EARLY

```bash
sudo -l
```

If you see something like:

```text
(ALL) NOPASSWD: /path/to/program
```

investigate that program.

### Workflow

```text
sudo -l
   ↓
What can I run?
   ↓
Can it execute commands / read-write files / spawn a shell?
   ↓
Check GTFOBins / known abuse method
   ↓
Run in authorized lab
   ↓
whoami
id
```

### Important

Do not memorize every `sudo` escape.

Memorize:

> **`sudo -l` → identify allowed binary → research abuse → execute → verify.**

---

# 6. SUID

Find SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Then:

```text
Is it unusual?
Is it writable?
Is it known to allow command execution?
Can GTFOBins provide an abuse path?
```

### Mental model

```text
SUID binary
    ↓
Runs with owner's privileges
    ↓
Owner = root?
    ↓
Can binary be abused?
    ↓
Potential root shell
```

---

# 7. PATH HIJACKING

Look for privileged programs/scripts that call commands without an absolute path.

Example concept:

```bash
service.sh
    ↓
calls: systemctl
    ↓
systemctl resolved through PATH
    ↓
attacker-controlled PATH entry
    ↓
malicious command executed
```

### Check

```bash
echo $PATH
```

Investigate scripts/programs that execute commands such as:

```text
cp
ls
tar
service
systemctl
```

without an absolute path.

### Required condition

```text
Privileged execution
+
Command resolved through controllable PATH
+
Writable PATH location
```

---

# 8. LINUX CAPABILITIES

Enumerate:

```bash
getcap -r / 2>/dev/null
```

Pay attention to capabilities such as:

```text
cap_setuid+ep
```

### Mental model

```text
Interesting capability
        ↓
What does the capability permit?
        ↓
Is the binary privileged / exploitable?
        ↓
Research known abuse
        ↓
Verify
```

---

# 9. CRON JOBS

Look for scheduled jobs.

Useful checks include:

```bash
cat /etc/crontab
ls -la /etc/cron.*
```

Also inspect scripts referenced by cron.

### High-value condition

```text
Cron runs as root
      +
Script/binary is writable
      ↓
Potential root execution
```

Check:

```bash
ls -la /path/to/script
cat /path/to/script
```

---

# 10. NFS

Inspect exports:

```bash
cat /etc/exports
```

Look for dangerous configurations such as:

```text
no_root_squash
```

### Core concept

`no_root_squash` can allow root privileges from an NFS client to remain root when accessing the exported filesystem.

### Workflow

```text
NFS export
   ↓
Check export options
   ↓
no_root_squash?
   ↓
Can the authorized lab client mount/use it?
   ↓
Investigate privilege escalation
```

---

# 11. AUTOMATED ENUMERATION

Use multiple tools; don't rely on one.

### LinPEAS

Broad automated privilege-escalation enumeration.

### LinEnum

System/users/cron/SUID and other enumeration.

### LES

Linux Exploit Suggester — checks kernel information against known exploits.

### Linux Smart Enumeration

Adjustable verbosity.

### Linux Priv Checker

Automated checks for common privilege-escalation opportunities.

### Key rule

```text
Automated enumeration
        ≠
Complete enumeration
```

Use tools to **save time and find leads**, then manually validate.

---

# 12. `pspy` — DYNAMIC ENUMERATION

When static enumeration doesn't reveal enough:

```bash
./pspy64
```

`pspy` is useful for observing processes and commands, including short-lived activity.

### Look for

```text
UID=0
```

especially when it executes:

- Shell scripts
- Cron-related commands
- Files in writable locations
- Commands using suspicious/writable inputs

### Core workflow

```text
Run pspy
   ↓
Watch recurring activity
   ↓
Find UID=0 process
   ↓
Identify command/script
   ↓
Check file permissions
   ↓
Can current user modify/influence it?
   ↓
Potential escalation
```

---

# 13. PUBLIC EXPLOITS

If enumeration reveals a vulnerable version:

```bash
uname -r
uname -a
cat /etc/os-release
```

Then research:

```bash
searchsploit <software> <version>
```

### Workflow

```text
ENUMERATE
   ↓
Software + version
   ↓
RESEARCH
   ↓
CVE / public exploit
   ↓
EVALUATE
   ↓
Version / architecture / distro / requirements
   ↓
EXPLOIT
   ↓
VERIFY
```

### Never assume

```text
CVE exists
    ≠
exploit will work
```

Read the exploit and understand its requirements first.

---

# 14. KERNEL EXPLOITS

Check:

```bash
uname -r
uname -a
cat /etc/os-release
```

Research the exact kernel/version.

```bash
searchsploit <kernel/version>
```

### Caution

Kernel exploits can be unstable. Evaluate compatibility and possible impact before execution, even in a lab.

---

# 15. NON-KERNEL SOFTWARE

Do not focus only on the kernel.

Check:

```text
Installed software
Services
SUID programs
Privileged applications
```

A vulnerable privileged userland program can provide an easier escalation path.

---

# 16. IF YOU GET CREDENTIALS

Once credentials are found:

```text
What account?
Is it privileged?
Can I authenticate as it?
Does it unlock sudo / SSH / another service?
```

Then verify:

```bash
whoami
id
```

---

# 17. QUICK TRIAGE ORDER

When you first land on a Linux machine:

```text
1. whoami / id
       ↓
2. sudo -l
       ↓
3. uname -a / OS version
       ↓
4. ps aux / ps axjf
       ↓
5. SUID
       ↓
6. Capabilities
       ↓
7. Cron
       ↓
8. Writable files/directories
       ↓
9. Network/listening services
       ↓
10. Automated enumeration
       ↓
11. pspy
       ↓
12. Public exploit research
```

---

# 18. FINDING → QUESTION → ACTION

| Finding | Ask | Next action |
|---|---|---|
| `sudo -l` | What can I execute as root? | Research binary abuse |
| SUID binary | Can it execute/read/write as root? | Research/GTFOBins |
| Capability | What does it permit? | Research abuse |
| Root cron | Can I modify its script? | Validate permissions |
| Root process | What is it executing? | Inspect file/input |
| Writable root script | Is it repeatedly executed? | Validate execution context |
| Unusual service | What version/account? | Research |
| Kernel version | Known CVE? | `searchsploit` / research |
| Credentials | Which account? | Test authorized reuse |
| `pspy` root command | Can I influence it? | Inspect permissions/inputs |

---

# 19. VERIFY EVERY ESCALATION

Always finish with:

```bash
whoami
id
```

Do not confuse:

```text
Exploit executed
```

with:

```text
Privilege escalation succeeded
```

---

# 20. 30-SECOND MEMORY MODEL

```text
LINUX PRIVESC

IDENTITY
  whoami / id / sudo -l

SYSTEM
  uname / os-release / processes

FILES
  writable / SUID / configs / credentials

PRIVILEGES
  sudo / SUID / capabilities

SCHEDULED
  cron

DYNAMIC
  pspy

NETWORK
  ports / services

SOFTWARE
  versions / CVEs / searchsploit

AUTOMATION
  LinPEAS / LinEnum / LES / LSE / Linux Priv Checker

VERIFY
  whoami / id
```

---

# 21. GOLDEN RULE

> **Don't ask only "What exploit do I know?" Ask "What is privileged, and what can my current user influence?"**

