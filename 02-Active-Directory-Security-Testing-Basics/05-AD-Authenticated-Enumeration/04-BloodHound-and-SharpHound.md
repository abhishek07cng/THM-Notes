# 🩸 BloodHound and SharpHound

> **Topic:** AD Authenticated Enumeration
>
> **Focus:** Graph-based Active Directory enumeration and attack-path analysis.

---

# 📖 Overview

BloodHound changed Active Directory enumeration by representing relationships as a graph instead of isolated lists.

The supplied material summarises the concept with:

> **“Defenders think in lists. Attackers think in graphs.” — John Lambert.**

---

# 🧠 Why Graphs Matter

Traditional enumeration may produce separate lists of:

```text
Users
Groups
Computers
Permissions
Sessions
```

BloodHound connects these relationships.

```text
User
 ↓
Group
 ↓
Computer
 ↓
Local Admin
 ↓
Another User
 ↓
Privileged Group
```

This makes potential attack paths easier to understand.

---

# 🔄 Two-Stage Model

The supplied material describes a two-stage approach.

## Stage 1 — Enumeration

Collectors gather:

- User sessions
- Group memberships
- ACLs
- Delegation settings
- Domain relationships

The collected information can then be analysed offline.

## Stage 2 — Targeted Analysis

BloodHound can identify efficient paths toward objectives such as privilege escalation.

---

# 🚀 Modern Capabilities Mentioned

The supplied material mentions:

- AzureHound for Azure Entra ID environments.
- RBCD-related attack-path primitives.
- Improved risk scoring and prioritisation.
- Hybrid cloud/on-premises analysis.

---

# 🧩 BloodHound vs SharpHound

These are not the same component.

### BloodHound

The graph-based analysis and visualisation platform.

### SharpHound

The official BloodHound data collector written in C#.

SharpHound gathers the data that BloodHound later visualises.

---

# 📊 SharpHound Collects

The supplied material highlights:

- Group memberships
- Session data
- Access Control Lists (ACLs)
- Domain trusts
- Privileged relationships
- Local administrator relationships

---

# 🛠️ SharpHound Variants

## SharpHound.exe

Windows executable intended for standard enumeration on domain-joined Windows machines.

## AzureHound.ps1

PowerShell collector focused on Azure Entra ID environments.

## SharpHound.ps1

The supplied material identifies the PowerShell variant as deprecated in recent releases.

## BloodHound.py

Python-based collector useful from Linux systems.

It can authenticate using:

- Credentials
- NTLM hashes
- Kerberos tickets

and can output JSON or ZIP data compatible with BloodHound.

---

# ⚠️ Version Compatibility

The supplied material recommends matching:

```text
BloodHound version
        +
SharpHound version
```

for best results.

It also notes that BloodHound.py is widely used but is not officially supported by the BloodHound development team.

---

# 🖥️ Running SharpHound

Example from the supplied lab:

```cmd
.\SharpHound.exe --CollectionMethods All --Domain tryhackme.loc --ExcludeDCs
```

### Parameters

| Parameter | Meaning |
|---|---|
| `--CollectionMethods All` | Use all available collection methods |
| `--Domain tryhackme.loc` | Target the specified domain |
| `--ExcludeDCs` | Exclude Domain Controllers from collection |

---

# 🐍 Running BloodHound.py

The supplied lab uses:

```bash
bloodhound-python -u asrepuser1 -p qwerty123! -d tryhackme.loc -ns 10.211.12.10 -c All --zip
```

### Parameters

| Parameter | Meaning |
|---|---|
| `-u` | Username |
| `-p` | Password |
| `-d` | Domain |
| `-ns` | DNS server IP |
| `-c All` | All available collection methods |
| `--zip` | Compress output into a ZIP archive |

---

# 📦 Example Collection Results

The supplied lab reports:

```text
Found 1 domains
Found 1 domains in the forest
Found 2 computers
Found 32 users
Found 58 groups
Found 5 gpos
Found 15 ous
Found 19 containers
Found 0 trusts
```

The output is compressed into a ZIP archive for ingestion.

---

# 🛡️ Operational Security Considerations

The supplied material warns that SharpHound and BloodHound.py can trigger security alerts.

It mentions:

```text
--ExcludeDCs
```

to avoid querying Domain Controllers.

It also mentions collection methods such as:

```text
DCOnly
```

to limit interaction with sensitive systems.

The material additionally mentions using `runas` with:

```text
/netonly
```

in appropriate assessment scenarios.

> **Important:** Use these techniques only with explicit authorisation.

---

# 🌐 BloodHound Community Edition

The supplied lab uses BloodHound-CE as a web application.

The workflow is:

```text
Collector
   ↓
ZIP Output
   ↓
BloodHound-CE
   ↓
File Ingest
   ↓
Explore Graph
```

---

# 📥 Importing Collection Data

After generating the ZIP:

1. Open BloodHound-CE.
2. Go to **Administration**.
3. Find **File Ingest**.
4. Upload the ZIP archive.

---

# 🔎 Exploring the Graph

Open:

```text
Explore
```

BloodHound represents:

### Nodes

Examples:

```text
Users
Computers
Groups
```

### Edges

Represent relationships and permissions between objects.

---

# 🔍 Node Information

Selecting a node can reveal:

- Object information
- Sessions
- Group membership
- Local administrator privileges
- Execution privileges
- Outbound object control
- Inbound object control

---

# 🧪 Built-in Queries

BloodHound provides built-in queries.

The supplied workflow:

```text
Cypher
 ↓
Folder Icon
 ↓
Browse Prebuilt Queries
```

Example:

```text
All Domain Admins
```

---

# 🧭 Attack Path Discovery

Use:

```text
Pathfinding
```

Then select:

```text
Start Node
      ↓
End Node
      ↓
Edge Filters
      ↓
Run Search
```

If a path exists, BloodHound visualises the relationship chain.

---

# ⚖️ Benefits and Limitations

## Benefits

- Graph-based AD visibility.
- Clear attack-path visualisation.
- Useful privilege-escalation analysis.
- Helps identify risky relationships.
- Useful for both offensive and defensive security work.

## Limitations

The supplied material specifically notes that SharpHound collection can be noisy and may trigger AV/EDR alerts.

---

# 💡 Key Takeaways

- BloodHound focuses on relationships rather than isolated lists.
- SharpHound is the official BloodHound collector.
- BloodHound.py provides a Python-based collection option.
- BloodHound data can reveal users, groups, sessions, ACLs, trusts and privilege relationships.
- Collection data can be imported into BloodHound-CE.
- Pathfinding can expose relationships between a starting object and a target.
- Collection activity can be detected by security controls.
