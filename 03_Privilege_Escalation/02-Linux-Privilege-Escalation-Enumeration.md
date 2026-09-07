# Linux Privilege Escalation — Enumeration

> **Revision Notes:** A quick-reference version of the supplied TryHackMe-style notes. Commands are grouped by enumeration objective, with the key reason for checking each item.

## 0. Privilege Escalation — Core Idea

**Privilege escalation** means moving from a lower-permission account to a higher-privileged account by exploiting a vulnerability, design flaw, or configuration oversight.

### Why it matters
- Initial access commonly provides a low-privileged account.
- Escalation can provide administrator/root-level capabilities.
- Higher privileges may allow password changes, access-control bypass, configuration changes, persistence, user privilege changes, and administrative commands.

### Revision mindset
Think of enumeration as answering: **Who am I? What system am I on? What is running? What users exist? What is exposed? What files can I access or modify? What privileged actions are available?**

---

## 1. OS Enumeration

### Goal

Identify the host, kernel, OS version, running processes, scheduled jobs, and installed software.

### Quick commands

```bash
hostname
uname -a
cat /proc/version
cat /etc/issue
ps aux
ps axjf
cat /etc/crontab
dpkg -l
```

### hostname

`hostname` returns the hostname. It may reveal the system's role, e.g. a production SQL server.

### uname

`uname -a` provides system information, including the kernel version. Kernel information can be useful when investigating potential kernel vulnerabilities.

### procfs

`cat /proc/version` can reveal the kernel version and build/compiler information.

### etc/issue

`cat /etc/issue` commonly shows OS/version information, although it can be customized.

### Processes

`ps` shows processes. Useful variants:
- `ps aux` — processes from all users; includes the launching user and processes without a terminal.
- `ps axjf` — all users + no controlling terminal + jobs format + process tree.

### Cron

Cron schedules commands/scripts. During enumeration, check privileged jobs and the files/scripts they execute.

```bash
cat /etc/crontab
ls -la /var/spool/cron/
ls -la /etc/cron.d/
```

Example:
```text
30 2 * * 1 root /home/ubuntu/clear-mail.sh
```
This means the script runs every Monday at 2:30 AM as `root`.

### Crontab fields

| Field | Meaning |
|---|---|
| 1 | Minute (0–59) |
| 2 | Hour (0–23) |
| 3 | Day of month (1–31) |
| 4 | Month (1–12) |
| 5 | Day of week (0–7; 0 and 7 = Sunday) |
| 6 | User running the task |
| 7 | Command/script/binary |

### dpkg

`dpkg -l` lists installed packages and their versions. This can help identify installed software and investigate known vulnerable binaries.

## 2. User Enumeration

### Goal

Identify the current user's privileges, environment, history, sudo rights, and local accounts.

### Quick commands

```bash
id
env
history
sudo -l
cat /etc/passwd
cat /etc/passwd | cut -d ":" -f 1
cat /etc/passwd | grep /home
```

### id

`id` provides the current user's UID, GID, and group memberships. It can also query another user, e.g. `id matt`.

### env

`env` displays environment variables. Pay particular attention to variables such as `PATH`; available interpreters or compilers may be relevant to later investigation.

### history

`history` shows previous commands. It can sometimes expose useful information such as usernames or passwords.

### sudo -l

`sudo -l` lists commands the current user is allowed to run through `sudo`. Depending on configuration, a password may be required.

### /etc/passwd

`/etc/passwd` is a useful source for discovering local users.

### Finding likely human users

`cat /etc/passwd | grep /home` can narrow the output to accounts with home directories under `/home`.

## 3. Network Enumeration

### Goal

Understand interfaces, connections, listening services, and possible network pivoting opportunities.

### Quick commands

```bash
ifconfig
ip addr
netstat -a
netstat -at
netstat -au
netstat -l
netstat -lt
netstat -tp
netstat -tpln
netstat -i
netstat -ano
```

### ifconfig / ip addr

`ifconfig` displays network interfaces. On modern Linux systems, `ip addr` is the modern equivalent. Interfaces such as `docker0` can indicate technologies running on the host.

### netstat basics

- `netstat -a` — all listening ports and established connections.
- `netstat -at` — TCP connections.
- `netstat -au` — UDP connections.
- `netstat -l` — listening ports.
- `netstat -lt` — listening TCP ports.
- `netstat -tp` — connections with service/PID information when available.
- `netstat -tpln` — listening TCP services, numeric addresses/ports, and process information where permitted.
- `netstat -i` — interface statistics.
- `netstat -ano` — all sockets, numeric addresses, and timers.

### Modern replacement

`ss` has largely replaced `netstat` on modern Linux distributions. The supplied notes give `ss -tpl` as an equivalent to `netstat -tpl`.

### Interpretation tip

Pay attention to whether a service binds to all interfaces or only localhost. The supplied examples include services listening on `0.0.0.0` and others on `127.0.0.1`.

## 4. File Enumeration

### Goal

Find files/directories that may reveal information or have permissions relevant to privilege escalation.

### ls

Use `ls -la` when inspecting directories so hidden files are included.

```bash
ls
ls -l
ls -la
```

### find — common searches

```bash
find . -name flag1.txt
find /home -name flag1.txt
find / -type d -name config
find / -type f -perm 0777
find / -perm -a=x
find /home -user frank
find / -mtime -10
find / -atime -10
find / -cmin -60
find / -amin -60
find / -size +50M
```

### Suppress permission errors

Use `2>/dev/null` when broad searches produce many permission-denied messages.

### Writable directories

The supplied notes give these examples:

```bash
find / -writable -type d 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -perm -o w -type d 2>/dev/null
```

### find -perm reminder

- `-perm mode` — exact permission match.
- `-perm -mode` — all specified permission bits must be set.
- `-perm /mode` — any specified permission bit may be set.

### World-executable directories

`find / -perm -o x -type d 2>/dev/null` finds world-executable directories.

### Development tools/languages

```bash
find / -name perl*
find / -name python*
find / -name gcc*
```

### Password-related filenames

`find / -name pass*.txt` can find filenames such as `pass.txt`, `password.txt`, and `passwords.txt`.

### SUID files

The SUID bit causes an executable to run with the privilege level of its owner rather than the invoking user.

```bash
find / -perm -u=s -type f 2>/dev/null
```

---

## 5. Fast Revision Checklist

### Step 1 — Identify the host
- [ ] `hostname`- [ ] `uname -a`- [ ] `cat /proc/version`- [ ] `cat /etc/issue`
### Step 2 — Understand the current user
- [ ] `id`- [ ] `env`- [ ] `history`- [ ] `sudo -l`- [ ] `/etc/passwd`
### Step 3 — Inspect processes and scheduled execution
- [ ] `ps aux`- [ ] `ps axjf`- [ ] `/etc/crontab`- [ ] `/etc/cron.d/`- [ ] `/var/spool/cron/`
### Step 4 — Inspect the network
- [ ] Interfaces: `ifconfig` / `ip addr`- [ ] Listening services: `netstat -lt` / `ss`- [ ] Service/PID mapping: `netstat -tpln`- [ ] Look at bind addresses such as `0.0.0.0` vs `127.0.0.1`
### Step 5 — Inspect files
- [ ] `ls -la`- [ ] Search for interesting filenames- [ ] Search for writable directories/files- [ ] Search for SUID files- [ ] Look for development tools/interpreters
## 6. Key Things to Remember

| Area | Remember |
|---|---|
| OS | Hostname + OS version + kernel |
| User | UID/GID + groups + sudo rights |
| Environment | `PATH` and other variables |
| Processes | What is running and as whom |
| Cron | What runs, when, and as which user |
| Packages | Installed software and versions |
| Network | Interfaces + listening services + bind addresses |
| Files | Hidden files + permissions + ownership |
| SUID | Executable may run with owner's privileges |

## 7. Mental Model

```text
INITIAL LOW-PRIVILEGE ACCESS
          │
          ▼
   ┌───────────────┐
   │ OS Enumeration│
   └───────┬───────┘
           ▼
   ┌────────────────┐
   │ User Enumeration│
   └───────┬────────┘
           ▼
   ┌────────────────┐
   │ Network         │
   │ Enumeration     │
   └───────┬────────┘
           ▼
   ┌────────────────┐
   │ File Enumeration│
   └───────┬────────┘
           ▼
   IDENTIFY INTERESTING
   CONFIGURATION / PERMISSIONS
           │
           ▼
   INVESTIGATE THE RELEVANT
   PRIVILEGE-ESCALATION PATH
```

> **Revision rule:** Do not memorize commands in isolation. Remember **what question each command answers** and what finding would make you investigate further.

## 8. Source Scope

This revision guide is reorganized from the supplied notes only. It does not add external techniques or claims beyond the source material.