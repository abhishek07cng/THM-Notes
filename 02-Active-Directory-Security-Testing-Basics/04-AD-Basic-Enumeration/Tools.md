# 🛠️ AD Basic Enumeration — Tools

> **Topic:** AD Basic Enumeration
>
> **Purpose:** Central reference for tools used and mentioned throughout the topic.

---

# 🧰 Tool Summary

| Tool | Category | Main Purpose |
|---|---|---|
| **fping** | Host Discovery | Discover live hosts across a subnet |
| **Nmap** | Network Enumeration | Host discovery, port scanning, service/version detection and SMB share enumeration |
| **smbclient** | SMB | List, browse, upload and download SMB-share contents |
| **smbmap** | SMB Enumeration | Enumerate SMB shares and permissions |
| **enum4linux / enum4linux-ng** | Windows Enumeration | Automate SMB/RPC/domain enumeration |
| **ldapsearch** | LDAP | Query Active Directory through LDAP |
| **rpcclient** | RPC | Test null sessions and enumerate users/domain information |
| **Kerbrute** | Kerberos Enumeration | Validate and enumerate Active Directory usernames |
| **CrackMapExec** | Network / SMB | Enumerate password policy and test credentials / password spraying |
| **Impacket smbclient** | SMB | Python-based SMB client from Impacket |

---

# 1️⃣ fping

## Purpose

Discovers live hosts across a subnet using ICMP.

## Command

```bash
fping -agq 10.211.11.0/24
```

### Options

| Option | Purpose |
|---|---|
| `-a` | Show alive systems |
| `-g` | Generate targets from the subnet |
| `-q` | Quiet mode |

---

# 2️⃣ Nmap

## Purpose

Used throughout the topic for:

- Host discovery
- Port scanning
- Service detection
- NSE scripts
- SMB share enumeration

## Host Discovery

```bash
nmap -sn 10.211.11.0/24
```

## AD Service Scan

```bash
nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP
```

## Scan Hosts from a File

```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
```

## Full TCP Port Scan

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

## SMB Share Enumeration

```bash
nmap -p445 --script smb-enum-shares 10.211.11.10
```

---

# 3️⃣ smbclient

## Purpose

Command-line SMB client used to:

- List shares
- Browse shares
- Download files
- Upload files

## List Shares Anonymously

```bash
smbclient -L //10.211.11.10 -N
```

## Connect to a Share

```bash
smbclient //10.211.11.10/SharedFiles -N
```

## Browse

```text
smb: \> ls
```

## Download

```text
smb: \> get Mouse_and_Malware.txt
```

## Exit

```text
smb: \> exit
```

---

# 4️⃣ smbmap

## Purpose

Enumerates SMB shares and shows permissions.

## Example

```bash
./smbmap.py -H 10.211.11.10
```

Useful for quickly identifying:

```text
READ
WRITE
NO ACCESS
```

---

# 5️⃣ enum4linux / enum4linux-ng

## Purpose

Automates Windows and SMB/RPC enumeration.

Can gather:

- Users
- Groups
- Shares
- Password policy
- RID information
- OS information
- NetBIOS information

## enum4linux

```bash
enum4linux -a TARGET_IP
```

## enum4linux-ng

```bash
enum4linux-ng -A 10.211.11.10 -oA results.txt
```

### Options

```text
-A
```

All available enumeration functions.

```text
-oA
```

Writes results to YAML and JSON.

---

# 6️⃣ ldapsearch

## Purpose

Queries LDAP and can be used to test anonymous LDAP binds.

## Test Anonymous Bind

```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```

## Query Users

```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```

### Options

| Option | Purpose |
|---|---|
| `-x` | Simple authentication / anonymous authentication |
| `-H` | LDAP server |
| `-s base` | Query only the base object |
| `-b` | Search base |

---

# 7️⃣ rpcclient

## Purpose

Used to interact with Windows RPC services.

Can be useful for:

- Null-session testing
- User enumeration
- Password-policy enumeration
- RID queries

## Test Null Session

```bash
rpcclient -U "" 10.211.11.10 -N
```

## Enumerate Users

```text
rpcclient $> enumdomusers
```

## Password Policy

```text
rpcclient $> getdompwinfo
```

## RID Query

```text
queryuser RID
```

## RID Cycling

```bash
for i in $(seq 500 2000); do echo "queryuser $i" | rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
```

---

# 8️⃣ Kerbrute

## Purpose

Validates candidate usernames using Kerberos.

## Installation

Download a suitable release binary, rename it to:

```text
kerbrute
```

then:

```bash
chmod +x kerbrute
```

## Username Enumeration

```bash
./kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users.txt
```

### Parameters

| Parameter | Purpose |
|---|---|
| `userenum` | Username enumeration |
| `--dc` | Domain Controller |
| `-d` | AD domain |
| `users.txt` | Candidate username list |

---

# 9️⃣ CrackMapExec

## Purpose

Network service exploitation and enumeration tool used in this topic for:

- SMB enumeration
- Password-policy enumeration
- Credential testing
- Password spraying

## Password Policy

```bash
crackmapexec smb 10.211.11.10 --pass-pol
```

## Password Spraying

```bash
crackmapexec smb 10.211.11.20 -u users.txt -p passwords.txt
```

---

# 🔟 Impacket smbclient

## Purpose

Python-based SMB client included in the Impacket toolkit.

The source identifies the AttackBox examples directory as:

```text
/opt/impacket/examples/
```

---

# 🔄 Tool Workflow

```text
fping / Nmap
      ↓
Host Discovery
      ↓
Nmap
      ↓
Domain Controller
      ↓
smbclient / smbmap
      ↓
SMB Shares
      ↓
ldapsearch / rpcclient
      ↓
Domain Information
      ↓
enum4linux-ng
      ↓
Candidate Users
      ↓
Kerbrute
      ↓
Valid Users
      ↓
rpcclient / CrackMapExec
      ↓
Password Policy
      ↓
CrackMapExec
      ↓
Password Spraying
```

---

# 📊 Tool-to-Technique Mapping

| Technique | Tool |
|---|---|
| Host Discovery | fping, Nmap |
| Port Scanning | Nmap |
| Service Detection | Nmap |
| SMB Share Enumeration | smbclient, smbmap, Nmap |
| SMB File Access | smbclient |
| LDAP Enumeration | ldapsearch |
| Automated Windows Enumeration | enum4linux-ng |
| RPC Null Session | rpcclient |
| RID Cycling | rpcclient, enum4linux-ng |
| Username Enumeration | Kerbrute |
| Password Policy Enumeration | rpcclient, CrackMapExec |
| Password Spraying | CrackMapExec |

---

# 🧠 Quick Revision

```text
fping
→ Find live hosts

Nmap
→ Find ports and services

smbclient
→ Access SMB shares

smbmap
→ See SMB permissions

ldapsearch
→ Query LDAP

enum4linux-ng
→ Automate Windows enumeration

rpcclient
→ Enumerate RPC/domain information

Kerbrute
→ Validate AD usernames

CrackMapExec
→ Query password policy + spray passwords

Impacket smbclient
→ Python SMB client
```

---

# 📌 Lab Reminder

The commands and values in this file come from the supplied TryHackMe material.

Examples such as:

```text
10.211.11.10
10.211.11.20
tryhackme.loc
```

are lab-specific and should only be used in an authorised environment.
