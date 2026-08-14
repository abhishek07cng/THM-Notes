# 🔍 Port Scanning and Domain Controller Identification

> **Topic:** AD Basic Enumeration
>
> **Section:** Mapping out the Network

---

# 📖 Overview

After identifying live hosts, the next objective is to determine which system is the **Domain Controller (DC)**.

The Domain Controller is especially important because it commonly exposes core Active Directory services.

---

# 🔑 Common Active Directory Ports

| Port | Protocol | What It Means |
|---:|---|---|
| `88` | Kerberos | Kerberos authentication and enumeration |
| `135` | MS-RPC | RPC enumeration and related Windows services |
| `139` | SMB/NetBIOS | Legacy SMB access |
| `389` | LDAP | LDAP queries to Active Directory |
| `445` | SMB | Modern SMB access; critical for enumeration |
| `464` | Kerberos / kpasswd | Password-related Kerberos service |
| `636` | LDAPS | Secure LDAP |

A host exposing several AD-specific services is a strong candidate for being a Domain Controller.

---

# 🛰️ Targeted Nmap Scan

Use:

```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
```

---

# 🔍 Command Breakdown

### `-p`

Specifies the ports to scan.

```text
88,135,139,389,445
```

### `-sV`

Enables service-version detection.

Nmap attempts to determine the versions of services running on open ports.

### `-sC`

Runs Nmap's default NSE scripts.

### `-iL`

Reads target hosts from:

```text
hosts.txt
```

---

# 🏢 Identifying the Domain Controller

A Domain Controller will often expose:

```text
88   Kerberos
389  LDAP
445  SMB
```

The Nmap output may also reveal:

- Windows Server information
- Domain names
- LDAP domain information
- SMB workgroup/domain information

For example:

```text
88/tcp  open  kerberos-sec
389/tcp open  ldap
445/tcp open  microsoft-ds
```

Together, these services strongly indicate an Active Directory environment.

---

# 🔬 Full Port Scan

When performing a more exhaustive assessment, scanning all TCP ports can prevent important services from being missed.

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

---

# 🔍 Full Scan Options

| Option | Meaning |
|---|---|
| `-sS` | TCP SYN scan |
| `-p-` | Scans all 65,535 TCP ports |
| `-T3` | Normal timing template |
| `-iL hosts.txt` | Reads targets from the host list |
| `-oN full_port_scan.txt` | Saves results to a normal output file |

---

# 🧭 Enumeration Flow

```text
Subnet
  ↓
Host Discovery
  ↓
Live Hosts
  ↓
Targeted Port Scan
  ↓
Kerberos + LDAP + SMB
  ↓
Domain Controller
  ↓
Domain Enumeration
```

---

# 💡 Key Takeaways

- Port scanning identifies services running on live hosts.
- Ports `88`, `389`, and `445` are especially useful when identifying AD infrastructure.
- Nmap `-sV` identifies service versions.
- Nmap `-sC` runs default NSE scripts.
- Nmap `-p-` scans all TCP ports.
- The Domain Controller is usually identifiable through its combination of AD-related services and service banners.
