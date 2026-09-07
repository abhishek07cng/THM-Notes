# Linux Privilege Escalation — Basics

> **Revision & Reference Notes** — Organized from the supplied notes for quick study, GitHub reference, and future write-ups.

## 0. Privilege Escalation — Quick Concept

Privilege escalation is the process of moving from a lower-privileged account to a higher-privileged account by exploiting a vulnerability, design flaw, or configuration weakness.

### Main idea
- Initial access often gives a low-privileged user.
- The goal is to identify a path to higher privileges.
- The path depends heavily on the target's configuration.

### Topics covered
1. Sudo
2. SUID / SGID
3. PATH hijacking
4. Linux capabilities
5. Cron jobs
6. NFS

---

## 1. Sudo Privilege Escalation

### Core concept
`sudo` can allow a regular user to execute specific programs with elevated privileges. Misconfigured sudo permissions can therefore provide a direct escalation path.

### First check
```bash
sudo -l
```

**Question to answer:** What commands can my current user execute through `sudo`, and as which user?

### Basic sudo abuse
Example:
```text
(ALL) NOPASSWD: /bin/cat
```

If `/bin/cat` can be executed as root, it can be used to read files that normally require root privileges, such as `/etc/shadow`.

### GTFOBins
Use **GTFOBins** to research whether an allowed binary has a known privilege-escalation technique.

Reference: https://gtfobins.github.io/

### Application-function abuse
Some binaries expose command-line functions that can be abused when the binary is allowed through sudo.

Example from the notes: Apache2 supports `-f` for an alternate configuration file and `-C` for processing a directive before reading configuration files.

```bash
apache2 -h
```

The supplied example demonstrates using an alternate configuration file to cause information from `/etc/shadow` to appear in an error message.

### LD_PRELOAD

Check whether sudo preserves `LD_PRELOAD`:

```text
Defaults env_keep+=LD_PRELOAD
```

Conceptual workflow:
1. Check `sudo -l` for `env_keep+=LD_PRELOAD`.
2. Create a shared library.
3. Compile it as a `.so` file.
4. Execute an allowed sudo program with `LD_PRELOAD` pointing to the library.
5. The library is loaded before the target program and can execute code with the elevated privileges.

Example compilation from the notes:
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

Example invocation:
```bash
sudo LD_PRELOAD=/home/user/ldpreload/shell.so find
```

> **Important:** The notes describe LD_PRELOAD as a relatively rare sudo vector. Always look for simpler sudo misconfigurations first.

---

## 2. SUID / SGID Privilege Escalation

### What is SUID?
SUID (Set-user Identification) causes an executable to run with the **effective UID of the file owner** rather than the UID of the user launching it.

### What is SGID?
SGID (Set-group Identification) similarly causes execution with the privileges of the file's group owner.

### Identify SUID/SGID files
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

Alternative focused searches:
```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

### What to look for
- SUID binary owned by `root`
- Unusual/custom SUID binaries
- SUID binaries with functionality that can read, write, or execute arbitrary resources
- Binary names that can be researched on GTFOBins

### Important distinction
**SUID alone does not automatically mean root access.** The binary must provide a useful capability or escape path.

### GTFOBins workflow
1. Find SUID binaries.
2. Identify unusual or interesting binaries.
3. Search the binary on GTFOBins.
4. Check the **SUID** technique rather than assuming the normal shell technique will work.

### SUID and shell privilege-drop trap
The supplied notes highlight an important issue: some shells detect a mismatch between real and effective UIDs and may drop privileges.

Useful checks:
```bash
readlink -f /bin/sh
echo $SHELL
```

`bash -p` can prevent Bash itself from dropping privileges in situations where Bash directly inherits the elevated effective UID.

However, an intermediate shell wrapper can drop privileges before `bash -p` is reached.

### More reliable approach: exec()
`execve()` / `execl()` replaces the current process image rather than spawning an intermediate wrapper shell.

Conceptually:
```text
SUID process retains EUID 0
        ↓
exec() replaces process
        ↓
new program inherits elevated privilege
```

### Vim payload progression from the notes
```vim
:!/bin/sh
:!/bin/bash -p
:shell
:py3 import os; os.execl('/bin/bash', 'bash', '-p')
:python import os; os.execl('/bin/bash', 'bash', '-p')
:perl exec '/bin/bash', '-p';
:lua os.execute('/bin/bash -p')
```

These depend on the Vim build and its available scripting features.

Check Vim features with:
```bash
vim --version
```

Look for features such as `+python3`, `+perl`, or `+lua`.

### General SUID methodology
```text
Find SUID binary
      ↓
Check GTFOBins
      ↓
Try canonical SUID technique
      ↓
Did privilege persist?
      ↓
YES → verify with id/whoami
NO  → suspect shell self-drop
      ↓
Check native exec()/embedded interpreter
      ↓
Verify effective UID
```

---

## 3. PATH Hijacking

### Core concept
`PATH` tells Linux which directories to search when an executable is referenced without an absolute path.

Example:
```bash
echo $PATH
```

If a **privileged program** executes a command by name rather than using an absolute path, and a user-controlled writable directory appears earlier in `PATH`, the user may be able to place a malicious replacement there.

### Questions to ask
1. Which directories are in `$PATH`?
2. Can the current user write to any of them?
3. Can `$PATH` be modified?
4. Is there a privileged script/program that executes a command by name?

### Find writable directories
```bash
find / -type d -writable 2>/dev/null | sort -u
```

### Conceptual attack chain
```text
Privileged SUID program
        ↓
system("thm")
        ↓
Searches $PATH
        ↓
Writable directory appears first
        ↓
Attacker-controlled 'thm'
        ↓
Command executes with privileged context
```

### PATH manipulation
Example from the notes:
```bash
export PATH=/tmp:$PATH
```

The notes then demonstrate placing a program named `thm` in `/tmp` and executing the vulnerable privileged program.

> **Key point:** PATH hijacking requires the right combination of a privileged program, command lookup by name, and a writable PATH directory.

---

## 4. Linux Capabilities

### Core concept
Linux capabilities split traditional root privileges into more granular permissions. A program can therefore receive a specific privileged capability without being fully SUID/root.

### Enumerate capabilities
```bash
getcap -r / 2>/dev/null
```

### Important capability
`cap_setuid+ep` allows a binary to change its UID, including changing to UID 0 (root), according to the supplied notes.

Example finding:
```text
/home/john/vim = cap_setuid+ep
```

### Why SUID enumeration can miss this
A binary with capabilities does **not** need the SUID bit.

Therefore:

```text
SUID enumeration ≠ complete privilege enumeration
                         ↓
Also check Linux capabilities
```

### Capability workflow
1. Run `getcap -r / 2>/dev/null`.
2. Identify unusual binaries with useful capabilities.
3. Research the binary and capability combination on GTFOBins.
4. Determine whether the capability allows UID manipulation or another privileged action.

### Example from the notes
The supplied example uses a Vim binary with `cap_setuid+ep` and its Python support to set UID 0 and execute a shell.

```bash
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

---

## 5. Cron Job Privilege Escalation

### Core concept
Cron runs commands/scripts at scheduled times. A cron job normally executes with the privileges of its configured owner.

### Vulnerable condition
```text
Privileged cron job
       +
User can modify executed script/binary
       ↓
User-controlled code runs with privileged context
```

### Main file
```bash
cat /etc/crontab
```

Other useful locations:
```text
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
/var/spool/cron/crontabs/
```

### Crontab format
```text
* * * * * user-name command
│ │ │ │ │       │      │
│ │ │ │ │       │      └─ Command
│ │ │ │ │       └──────── User
│ │ │ │ └──────────────── Day of week
│ │ │ └────────────────── Month
│ │ └──────────────────── Day of month
│ └────────────────────── Hour
└──────────────────────── Minute
```

### Example
```text
* * * * * root /home/john/Desktop/backup.sh
```

This runs `backup.sh` every minute as root.

### What to inspect
- Is the job executed as root?
- Is the script path known?
- Can the script be modified?
- Is the referenced file missing?
- Does the cron environment use a writable directory in `PATH`?
- Does the script invoke utilities in a way that introduces another weakness?

### Writable script
If a root cron job executes a script that the current user can modify, the modified script can execute with root privileges.

The notes demonstrate using a reverse shell in a lab context and receiving the resulting privileged connection.

### Missing script / PATH-based cron issue
Example:
```text
* * * * * root antivirus.sh
```

If `antivirus.sh` cannot be found and the cron `PATH` includes a directory writable by the current user, the user may be able to create a malicious `antivirus.sh` there.

### Additional cron attack surface
The supplied notes also mention tools such as `tar`, `7z`, and `rsync`, whose command-line/wildcard behaviour may create additional attack paths depending on how a privileged scheduled job uses them.

> **Revision rule:** Always inspect both the **cron entry** and the **file/command it executes**. A cron job by itself is not necessarily vulnerable.

---

## 6. NFS Privilege Escalation

### Core concept
NFS (Network File System) can expose directories to remote systems. Misconfigured exports can allow a user to create files that retain root ownership/privileges.

### Configuration file
```bash
cat /etc/exports
```

### Critical option: `no_root_squash`
Normally, NFS can map remote root to `nfsnobody` to prevent remote root privileges from being preserved.

`no_root_squash` disables that protection.

### Dangerous combination
```text
Writable NFS share
       +
no_root_squash
       ↓
Files created through the share can retain root ownership
       ↓
Potential SUID root executable
       ↓
Privilege escalation
```

### Enumerate exported shares
From the attacking machine:
```bash
showmount -e MACHINE_IP
```

### Mount a share
Example from the supplied notes:
```bash
mkdir /tmp/backupsonattackermachine
mount -o rw MACHINE_IP:/backups /tmp/backupsonattackermachine
```

### What to verify
- Is the share writable?
- Is `no_root_squash` enabled?
- Can files be created with root ownership?
- Can SUID permissions be preserved?

The supplied lab demonstrates compiling a binary on the mounted share, setting the SUID bit, and then executing it from the target where it appears as a root-owned SUID executable.

---

## 7. Privilege Escalation — Master Checklist

### Sudo
- [ ] `sudo -l`- [ ] Identify `NOPASSWD` entries- [ ] Check which user commands run as- [ ] Research allowed binaries on GTFOBins- [ ] Check for dangerous environment preservation such as `LD_PRELOAD`
### SUID / SGID
- [ ] Search SUID files- [ ] Search SGID files- [ ] Identify unusual/custom binaries- [ ] Check GTFOBins- [ ] Watch for shell privilege-drop behaviour
### PATH
- [ ] `echo $PATH`- [ ] Identify writable PATH directories- [ ] Find privileged programs using relative command names- [ ] Check whether PATH can be modified
### Capabilities
- [ ] `getcap -r / 2>/dev/null`- [ ] Look for powerful capabilities such as `cap_setuid`- [ ] Check binaries with capabilities on GTFOBins
### Cron
- [ ] `/etc/crontab`- [ ] `/etc/cron.d/`- [ ] `/etc/cron.*`- [ ] `/var/spool/cron/crontabs/`- [ ] Identify privileged jobs- [ ] Check script/binary permissions- [ ] Check missing commands and PATH
### NFS
- [ ] `cat /etc/exports`- [ ] `showmount -e MACHINE_IP`- [ ] Identify writable shares- [ ] Check for `no_root_squash`- [ ] Determine whether root-owned/SUID files can be created
---

## 8. What to Memorize vs. What to Look Up

### Memorize
| Concept | Essential memory |
|---|---|
| Sudo | `sudo -l` |
| SUID | `find / -perm -4000 -type f 2>/dev/null` |
| Capabilities | `getcap -r / 2>/dev/null` |
| PATH | `echo $PATH` + writable directories |
| Cron | `/etc/crontab` + `/etc/cron.d/` + permissions |
| NFS | `/etc/exports` + `showmount -e` + `no_root_squash` |
| GTFOBins | Check the binary and the specific technique |

### Look up when needed
- Exact GTFOBins payloads- Binary-specific escape techniques- Available interpreter features- Tool-specific command syntax- Exact exploit chains for a particular target

---

## 9. Fast Decision Tree

```text
LOW-PRIVILEGED SHELL
        │
        ├── sudo -l        │     └── Allowed privileged binary?        │            └── Check GTFOBins / configuration        │
        ├── SUID/SGID        │     └── Interesting binary?        │            └── Check GTFOBins / privilege-drop issues        │
        ├── PATH        │     └── Writable PATH directory + privileged command lookup?        │
        ├── Capabilities        │     └── Powerful capability on unusual binary?        │
        ├── Cron        │     └── Root job + writable script/path?        │
        └── NFS              └── Writable export + no_root_squash?```

## 10. Final Revision Principle

> **Enumerate first, understand the condition, then choose the technique.**

Do not treat every SUID binary, cron job, sudo rule, capability, PATH entry, or NFS export as automatically exploitable. The important skill is recognizing the **combination of conditions** that creates a privilege-escalation path.

## Source Scope

This document is reorganized from the supplied notes. Lab credentials, machine IPs, and other temporary VM details were intentionally omitted from the revision-focused reference where they are not necessary for learning the underlying concept.