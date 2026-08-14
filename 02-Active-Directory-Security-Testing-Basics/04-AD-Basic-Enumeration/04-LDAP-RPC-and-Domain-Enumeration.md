# 🏢 LDAP, RPC and Domain Enumeration

> **Topic:** AD Basic Enumeration
>
> **Section:** Domain Enumeration

---

# 📖 Overview

After identifying the Domain Controller and enumerating exposed SMB services, the next step is to enumerate the domain itself.

Important sources include:

- LDAP
- RPC
- SMB
- Kerberos

This section focuses on anonymous LDAP binds, enum4linux-ng and RPC null sessions.

---

# 🗂️ LDAP Enumeration

**Lightweight Directory Access Protocol (LDAP)** is used to access and manage directory services such as Microsoft Active Directory.

LDAP can expose information about:

- Users
- Groups
- Devices
- Organisational information
- Domain structure

---

# 👤 Anonymous LDAP Bind

Some LDAP servers allow anonymous users to perform read-only queries.

This can expose useful directory information without valid credentials.

Test anonymous LDAP access with:

```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```

---

# 🔍 Command Breakdown

| Option | Meaning |
|---|---|
| `-x` | Simple authentication; used here for anonymous authentication |
| `-H` | Specifies the LDAP server |
| `-s base` | Limits the query to the base object |

---

# 📄 Example Information

A successful query may reveal:

```text
rootDomainNamingContext: DC=tryhackme,DC=loc
dnsHostName: DC.tryhackme.loc
defaultNamingContext: DC=tryhackme,DC=loc
configurationNamingContext: CN=Configuration,DC=tryhackme,DC=loc
```

This can reveal:

- Domain name
- Domain Controller hostname
- Naming contexts
- AD configuration structure

---

# 👥 Querying User Information

Once anonymous access is confirmed:

```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```

This queries objects whose class is:

```text
person
```

and can return user-related directory information.

---

# 🧰 enum4linux-ng

`enum4linux-ng` automates multiple enumeration techniques against Windows systems.

It can gather information such as:

- Users
- Groups
- Shares
- Password policy
- RID information
- OS information
- NetBIOS information

---

# 🔎 Full Enumeration

```bash
enum4linux-ng -A 10.211.11.10 -oA results.txt
```

### Options

```text
-A
```

Performs all available enumeration functions.

```text
-oA
```

Writes results to YAML and JSON files.

---

# 🔄 RPC Enumeration

Microsoft Remote Procedure Call (MSRPC) allows programs to request services from remote systems.

RPC services can be accessed through SMB.

When SMB permits **null sessions**, unauthenticated users may connect to:

```text
IPC$
```

and enumerate:

- Users
- Groups
- Shares
- Other system/domain information

---

# 🔓 Testing an RPC Null Session

```bash
rpcclient -U "" 10.211.11.10 -N
```

### Options

```text
-U ""
```

Uses an empty username for anonymous access.

```text
-N
```

Prevents a password prompt.

---

# 👤 Enumerating Domain Users

If the null session succeeds:

```text
rpcclient $> enumdomusers
```

Example output:

```text
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[gerald.burgess] rid:[0x650]
user:[nigel.parsons] rid:[0x651]
user:[guy.smith] rid:[0x652]
```

These usernames can later be validated with Kerbrute.

---

# 🆘 rpcclient Help

Inside the `rpcclient` shell:

```text
help
```

can be used to view available commands.

With sufficient permissions, RPC can provide extensive domain information.

---

# 🆔 RID Enumeration

Every Active Directory object has a Security Identifier (SID).

The SID contains a:

```text
Relative Identifier (RID)
```

that uniquely identifies the object within the domain.

---

# 📊 Well-Known RIDs

The source identifies:

| RID | Object |
|---:|---|
| `500` | Administrator |
| `501` | Guest |
| `512` | Domain Admins |
| `513` | Domain Users |
| `514` | Domain Guests |

User accounts typically begin around:

```text
RID 1000+
```

---

# 🔄 RID Cycling

If normal user enumeration is restricted, individual RIDs can be queried.

The source provides:

```bash
for i in $(seq 500 2000); do echo "queryuser $i" | rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
```

---

# 🔍 Command Breakdown

```text
for i in $(seq 500 2000)
```

Iterates through possible RIDs.

```text
queryuser $i
```

Queries the object associated with the current RID.

```text
2>/dev/null
```

Suppresses error output.

```text
grep -i "User Name"
```

Displays lines containing `User Name`, ignoring case.

> **Lab Note:** The source notes that this command may take approximately 2–3 minutes to complete.

---

# 💡 Key Takeaways

- LDAP can reveal important AD structure.
- Anonymous LDAP binds may expose directory information.
- `ldapsearch` can test anonymous LDAP access.
- `enum4linux-ng` automates several enumeration methods.
- RPC null sessions can expose users and other information.
- `rpcclient` can enumerate domain users.
- RID cycling can recover user information when direct enumeration is restricted.
