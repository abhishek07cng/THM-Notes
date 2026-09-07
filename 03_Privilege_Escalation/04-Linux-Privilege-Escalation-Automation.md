# Linux Privilege Escalation — Automation & Public Exploits

> **Purpose:** Revision and reference notes for Linux privilege escalation.  
> **Focus:** Automating enumeration, researching public exploits, and discovering short-lived privileged processes with `pspy`.

---

## 1. Automated Enumeration Tools

### Why automate enumeration?

Manual enumeration is thorough but time-consuming. Automated tools can quickly highlight common privilege-escalation paths such as:

- Misconfigurations
- Weak file permissions
- Credentials
- SUID binaries
- Scheduled tasks
- Kernel/software vulnerabilities
- Other potentially interesting system information

**Important:** Automated tools are helpers, not replacements for manual enumeration. They can miss privilege-escalation vectors.

### Tool-selection principle

The target's environment determines which tools you can run.

For example, a Python-based tool is not useful if Python is unavailable on the target.

**Best practice:** Know several enumeration tools rather than depending on one tool.

### Common tools

| Tool | Main purpose |
|---|---|
| **LinPEAS** | Highlights privilege-escalation paths, including misconfigurations, weak permissions, credentials, and more |
| **LinEnum** | Collects system information, users, cron jobs, SUID binaries, etc. in a readable report |
| **LES (Linux Exploit Suggester)** | Compares the kernel version against known CVEs and suggests potentially applicable local exploits |
| **Linux Smart Enumeration (LSE)** | Local enumeration with adjustable verbosity; starts quietly and reveals more detail at higher levels |
| **Linux Priv Checker** | Enumerates system information and checks for common privilege-escalation opportunities |

### Revision point

Think of automated enumeration as:

> **Fast coverage → identify leads → manually validate → exploit only when the condition is confirmed.**

---

# 2. Public Exploits

## Misconfiguration vs. Public Exploit

This distinction is important.

### Misconfiguration

The software itself may be working as intended, but it has been configured insecurely.

Examples from the notes:

- Overly permissive `sudo` rule
- Cron job executing a world-writable script
- Inappropriate capability assigned to a binary

### Public exploit

The **software itself contains a vulnerability**.

Examples of underlying bug classes mentioned:

- Buffer overflow
- Race condition
- Logic error

When vulnerabilities are discovered, they are assigned a **CVE (Common Vulnerabilities and Exposures)** identifier, and exploit code may be published publicly.

### Quick comparison

| Situation | Root cause |
|---|---|
| Weak `sudo` rule | Configuration |
| World-writable root cron script | Configuration |
| Dangerous capability assignment | Configuration |
| Vulnerable kernel/software | Software vulnerability |
| CVE with public exploit | Software bug + published exploit |

---

# 3. Public-Exploit Methodology

Do **not** treat public exploits as:

> Find code → run code → hope for root

Use a structured workflow.

## Step 1 — Enumerate

Identify:

- Installed software
- Software versions
- Kernel version
- Distribution/version
- SUID binaries
- Services running as root

Useful kernel/OS commands:

```bash
uname -r
uname -a
cat /etc/os-release
```

---

## Step 2 — Research

Take the information discovered during enumeration and search for known vulnerabilities.

Ask:

- Is there a CVE for this version?
- Is public exploit code available?
- Does the exploit apply to this exact software/version?
- Does it match the target's architecture/distribution?

### Common sources

#### `searchsploit`

`searchsploit` searches a local copy of the Exploit-DB database and comes pre-installed on Kali.

Basic usage:

```bash
searchsploit <software> <version>
```

#### GitHub

GitHub contains repositories with public exploit implementations.

If you identify a CVE, a useful research pattern is:

```text
CVE-<ID> github
```

#### Enumeration tools

Tools such as LinPEAS may identify vulnerable software and provide CVE numbers, giving you a starting point for further research.

---

# 4. Evaluate the Exploit Before Running It

Finding an exploit does **not** mean it will work.

Read and understand the exploit first.

### Check its requirements

| Question | Why it matters |
|---|---|
| Does it support the target version? | Avoid incompatible exploits |
| Does it require a particular kernel version? | Kernel exploits are often version-specific |
| Does it support the target architecture? | Binary/compiled exploits may depend on architecture |
| Does it require a particular distribution? | Environment differences can matter |
| Does it require `gcc` on the target? | Some exploits need compilation |
| Could it crash the system? | Exploits can be unstable |

### Key principle

> **A CVE match is a lead, not proof that exploitation will succeed.**

---

# 5. Exploit

Once the exploit has been evaluated:

1. Obtain the exploit in the authorized lab environment.
2. Transfer it to the target if necessary.
3. Compile it if required.
4. Execute it according to its requirements.

Example from the lab workflow:

```bash
scp <file> john@10.49.133.153:/home/john/
```

> The IP above is a lab example from the source notes; use the actual authorized target address in your own environment.

---

# 6. Verify Privilege Escalation

After exploitation, confirm that privileges actually changed.

Useful commands:

```bash
whoami
id
```

Then verify that you can access something that was previously unavailable to the low-privileged account.

### Verification mindset

Never assume:

> exploit executed successfully = root obtained

Instead:

> **Exploit → verify identity → verify privileges → continue**

---

# 7. Kernel Exploits

The Linux kernel operates with the highest level of privilege on the system.

Therefore, a kernel vulnerability can potentially allow:

```text
Unprivileged user
      ↓
Kernel vulnerability
      ↓
Root
```

### Enumerate kernel information

```bash
uname -r
uname -a
cat /etc/os-release
```

Then research known vulnerabilities for the identified version.

Example approach:

```bash
searchsploit <kernel/version>
```

or research the relevant version/CVE through trusted public sources.

### Important caution

Kernel exploits can be particularly risky because the kernel is central to system operation. The source notes emphasize evaluating exploits before execution rather than blindly running them.

---

# 8. Non-Kernel Public Exploits

Public privilege-escalation vulnerabilities are not limited to the kernel.

They can affect **userland software** — programs and utilities that may run with elevated privileges.

The notes highlight that these can often be:

- Easier to exploit
- Less likely to crash the system

### Mental model

```text
Kernel exploit
    ↓
Target kernel

Non-kernel exploit
    ↓
Vulnerable privileged userland software
```

---

# 9. Public Exploit Workflow — Quick Reference

```text
ENUMERATE
   ↓
Identify software + versions + kernel + SUID/services
   ↓
RESEARCH
   ↓
Find CVE / public exploit
   ↓
EVALUATE
   ↓
Read code + check requirements + compatibility + stability
   ↓
EXPLOIT
   ↓
Transfer / compile / execute
   ↓
VERIFY
   ↓
whoami + id + privilege/access check
```

---

# 10. `pspy` — Unprivileged Process Monitoring

## The problem with polling

Automated enumeration tools mainly provide a **snapshot**.

A snapshot can miss short-lived processes such as:

- Cron jobs
- Scheduled scripts
- Commands that execute and exit very quickly

For example:

```text
Process starts
   ↓
Runs for milliseconds
   ↓
Exits
```

A traditional enumeration command may never catch it.

---

## What is `pspy`?

`pspy` is a process-monitoring tool that allows an unprivileged user to observe:

- Running processes
- Cron jobs
- Commands executed by other users

It does this **without requiring root privileges**.

### Core idea

> Automated enumeration tells you what is present **now**; `pspy` helps reveal what happens **over time**.

---

# 11. `pspy` Event-Driven Approach

Linux process visibility can make short-lived tasks difficult to discover through ordinary polling.

`pspy` uses an **event-driven approach**.

According to the source notes, it:

1. Sets `inotify` watches on commonly accessed directories such as:
   - `/etc`
   - `/tmp`
   - `/usr`
   - `/var`
2. Detects filesystem activity.
3. Quickly scans `/proc`.
4. Identifies newly appearing processes.
5. Captures information such as:
   - UID
   - PID
   - Timestamp
   - Full command

### Why this matters

A process may exist only briefly, but its process metadata can be observed during its lifetime.

`pspy` does not bypass kernel permissions; it reacts quickly enough to catch activity that polling may miss.

---

# 12. Running `pspy`

In the lab environment described by the source:

```bash
./pspy64
```

The tool may reveal processes running with:

```text
UID=0
```

`UID=0` indicates a root-owned process.

### What to look for

Pay particular attention to:

```text
UID=0
```

combined with:

- Scripts executed from unusual locations
- Scripts in writable directories
- Commands using writable files
- Recurring/scheduled activity
- Root processes invoking shell scripts
- Commands whose arguments expose useful paths

---

# 13. Reading `pspy` Output

Example pattern from the source:

```text
CMD: UID=0 PID=1937 | /bin/bash /root/run-backup.sh
CMD: UID=0 PID=1938 | tar -czf /var/backup/syslog.tar.gz /var/log/syslog
CMD: UID=0 PID=1942 | /bin/bash /root/run-rm-tmp.sh
CMD: UID=0 PID=1943 | /bin/bash /usr/local/bin/rm-tmp.sh
```

The important observation is not simply:

> “A root process exists.”

Instead ask:

> **What root process is running, what file/script does it execute, and can my current user influence that file or its inputs?**

---

# 14. Root Cron / Scheduled Script Misconfiguration

The source gives this example:

```bash
ls -la /usr/local/bin/rm-tmp.sh
```

Output:

```text
-rwxrwxrwx 1 root root 57 Jan 20 10:27 /usr/local/bin/rm-tmp.sh
```

The script:

```bash
#!/bin/bash

rm -r /tmp/*
```

The critical issue is:

```text
World-writable script
        +
Executed as root
        ↓
Low-privileged user can modify root-executed code
```

This is a **misconfiguration-based privilege escalation**, not a software CVE.

---

# 15. Recognizing the Exploitation Condition

When `pspy` shows a root process executing a script, investigate:

### 1. Who owns the script?

```bash
ls -la /path/to/script
```

### 2. Can your user modify it?

Look at the permission bits.

For example:

```text
-rwxrwxrwx
```

means the file is writable by owner, group, and others.

### 3. Is it executed with elevated privileges?

`pspy` output can reveal:

```text
UID=0
```

### 4. Is it recurring?

Repeated execution makes a race/timing opportunity much easier to observe.

### Attack condition

```text
Root executes script
        +
Current user can modify script
        +
Script executes with root privileges
        ↓
Potential privilege escalation
```

---

# 16. Lab Example: Root Password Change

The source demonstrates adding a command to the root-executed script:

```bash
echo "root:newpass" | chpasswd
```

Then, in the lab:

```bash
su
```

and authenticate with the newly configured password.

### What this teaches

The important lesson is the **vulnerability pattern**, not memorizing one payload:

> **A privileged process executing user-writable code creates a privilege-escalation path.**

---

# 17. `pspy` Investigation Workflow

When automated enumeration does not reveal an obvious path:

```text
Run pspy
   ↓
Watch for recurring activity
   ↓
Look for UID=0 processes
   ↓
Identify executed scripts/commands
   ↓
Check file ownership + permissions
   ↓
Check whether your user can influence them
   ↓
Understand execution context
   ↓
Exploit only in the authorized lab
   ↓
Verify with whoami / id
```

---

# 18. Automation + Manual Enumeration

Do not replace the manual methodology from previous sections with automation.

Use both.

### Recommended sequence

```text
1. Manual enumeration
       ↓
2. Automated enumeration
       ↓
3. Compare / investigate findings
       ↓
4. Research public exploits
       ↓
5. Monitor dynamic activity with pspy
       ↓
6. Validate the attack condition
       ↓
7. Exploit
       ↓
8. Verify
```

### Why?

Each approach catches different things:

| Technique | Best at |
|---|---|
| Manual enumeration | Understanding the system and validating details |
| LinPEAS / LinEnum / etc. | Quickly identifying common static findings |
| Public-exploit research | Finding software-level vulnerabilities |
| `pspy` | Discovering dynamic/short-lived privileged activity |

---

# 19. High-Value Revision Table

| Finding | Question to ask |
|---|---|
| Vulnerable software version | Is there a matching CVE? |
| Kernel version | Are there known local privilege-escalation vulnerabilities? |
| SUID binary | Can it be abused to execute something as another user/root? |
| Root service | What software/version is it running? |
| Root cron/script | Is the executed file writable by my user? |
| `UID=0` process in `pspy` | What command/script is being executed? |
| Writable privileged script | Can I influence code executed as root? |
| Public exploit | Does it actually match the target environment? |

---

# 20. Commands to Memorize

### OS / kernel

```bash
uname -r
uname -a
cat /etc/os-release
```

### Exploit research

```bash
searchsploit <software> <version>
```

### Transfer in the lab

```bash
scp <file> <user>@<target>:/path/
```

### Privilege verification

```bash
whoami
id
```

### File investigation

```bash
ls -la /path/to/file
cat /path/to/file
```

### Process monitoring

```bash
./pspy64
```

---

# 21. What to Memorize vs. What to Look Up

## Memorize

- Manual enumeration should come before/alongside automation.
- Automated tools can miss findings.
- Know multiple enumeration tools.
- `uname -r`, `uname -a`, `/etc/os-release`
- `searchsploit`
- Public-exploit workflow:
  **Enumerate → Research → Evaluate → Exploit → Verify**
- `pspy` is useful for dynamic/short-lived process discovery.
- `UID=0` means the process is running as root.
- A writable script executed by root is a critical finding.
- Always verify privileges after exploitation.

## Look up when needed

- Exact CVE details
- Exact exploit code
- Exploit-specific compilation requirements
- Architecture compatibility
- Kernel-version compatibility
- Distribution-specific requirements
- `pspy` options and advanced usage

---

# 22. Fast Decision Tree

```text
Do I have a clear misconfiguration?
        │
      YES ──→ Validate permissions / execution context
        │
       NO
        ↓
Run automated enumeration
        ↓
Interesting software/version?
        │
      YES ──→ Research CVEs / searchsploit
        │
       NO
        ↓
Need to understand scheduled/dynamic activity?
        │
      YES
        ↓
Run pspy
        ↓
See UID=0 process?
        │
      YES
        ↓
What command/script is executed?
        ↓
Can I influence the file/input?
        │
      YES ──→ Validate → exploit in authorized environment
```

---

# 23. Core Mental Model

Privilege escalation automation is not about blindly running scripts.

Think in three layers:

### Layer 1 — Static

> **What is configured right now?**

Use:

- Manual enumeration
- LinPEAS
- LinEnum
- LSE
- Linux Smart Enumeration
- Linux Priv Checker

### Layer 2 — Software vulnerabilities

> **Is the installed software itself vulnerable?**

Use:

- Version enumeration
- CVE research
- `searchsploit`
- Public exploit repositories

### Layer 3 — Dynamic

> **What privileged activity happens after I start watching?**

Use:

- `pspy`

### Overall model

```text
STATIC CONFIGURATION
        +
SOFTWARE VULNERABILITIES
        +
DYNAMIC ACTIVITY
        ↓
Complete privilege-escalation picture
```

---

# 24. Final Revision Checklist

Before considering Linux privilege-escalation enumeration complete, ask:

- [ ] Did I identify the OS/distribution?
- [ ] Did I identify the kernel version?
- [ ] Did I enumerate users and privileges?
- [ ] Did I inspect SUID/capabilities and other common vectors?
- [ ] Did I use automated enumeration where appropriate?
- [ ] Did I avoid relying on only one automated tool?
- [ ] Did I identify interesting software and versions?
- [ ] Did I research matching CVEs?
- [ ] Did I evaluate exploit requirements before running code?
- [ ] Did I consider architecture/distribution compatibility?
- [ ] Did I monitor dynamic activity with `pspy` when useful?
- [ ] Did I investigate `UID=0` processes?
- [ ] Did I check permissions on root-executed scripts?
- [ ] Did I verify successful escalation with `whoami` and `id`?

---

## One-Line Takeaway

> **Automate to save time, research to find software vulnerabilities, use `pspy` to catch dynamic privileged activity, manually validate every lead, and always verify the final privilege level.**

---

## Source Scope

This revision guide is organized from the supplied **“04-Linux Privilege Escalation: Automation”** material. Lab-specific IPs, usernames, and temporary environment details have been kept only where they illustrate the technique; they should be replaced with the values of the authorized lab environment being used.
