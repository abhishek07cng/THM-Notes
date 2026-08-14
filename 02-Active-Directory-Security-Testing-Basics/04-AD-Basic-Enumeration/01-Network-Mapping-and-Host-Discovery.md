# 🌐 Network Mapping and Host Discovery

> **Topic:** AD Basic Enumeration
>
> **Section:** Mapping out the Network

---

# 📖 Overview

Active Directory enumeration is a crucial first step in penetration testing Microsoft Windows enterprise networks.

During many internal penetration tests, we may receive VPN access to the target network without user credentials.

The objective is therefore to gather as much information as possible about:

- Users
- Groups
- Computers
- Policies
- Network services

This information can reveal potential vulnerabilities and attack paths that may provide an initial foothold.

---

# 🗺️ Network Mapping

The first phase is identifying which hosts are alive in the target network.

```text
Target Subnet
      ↓
Host Discovery
      ↓
Live Hosts
      ↓
Port Scanning
      ↓
Identify Domain Controller
```

---

# 🔎 Host Discovery with fping

`fping` is similar to `ping`, but it can target multiple systems, including an entire subnet.

Instead of repeatedly probing one host, `fping` moves through the target list.

## Command

```bash
fping -agq 10.211.11.0/24
```

Example output:

```text
10.211.11.1
10.211.11.10
10.211.11.20
10.211.11.250
```

---

# 🔍 fping Options

| Option | Meaning |
|---|---|
| `-a` | Shows systems that are alive |
| `-g` | Generates a target list from an IP netmask |
| `-q` | Quiet mode; suppresses per-probe results and ICMP errors |

---

# 📝 Creating a Host List

Once live hosts have been identified, store relevant IP addresses in:

```text
hosts.txt
```

Example:

```bash
cat hosts.txt
```

```text
10.211.11.20
10.211.11.10
```

This file can later be supplied to Nmap.

> **Lab Note:** The source identifies gateway and VPN-server addresses as out-of-scope infrastructure that should be ignored when they are outside the intended testing scope.

---

# 🔎 Host Discovery with Nmap

Nmap can perform host discovery using ping-scan mode:

```bash
nmap -sn 10.211.11.0/24
```

### `-sn`

Performs a ping scan to determine which hosts are up without performing a port scan.

---

# 🧠 Why Host Discovery Matters

Before enumerating AD services, we need to know which systems are alive.

The resulting host list becomes the foundation for:

```text
Live Hosts
    ↓
Port Scan
    ↓
Service Detection
    ↓
Domain Controller Identification
```

---

# 💡 Key Takeaways

- Host discovery identifies live systems within the target range.
- `fping` can scan an entire subnet efficiently.
- Nmap's `-sn` mode performs host discovery without port scanning.
- Store relevant live hosts in `hosts.txt`.
- Always respect the defined penetration-testing scope.
