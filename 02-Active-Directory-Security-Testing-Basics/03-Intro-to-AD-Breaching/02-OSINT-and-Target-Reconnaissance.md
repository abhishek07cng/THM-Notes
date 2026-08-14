# 🔎 OSINT and Target Reconnaissance

> **Topic:** Introduction to AD Breaching
>
> **Section:** OSINT and Target Reconnaissance

---

# 📖 Overview

Before launching active attacks against an AD environment, build an initial picture of the target.

In a real engagement, this begins with **Open-Source Intelligence (OSINT)**.

The goal is:

> **Build a list of potential usernames that can be validated against the domain.**

---

# 🌐 Gathering Usernames From OSINT

Potential sources include:

## LinkedIn

Employee profiles can reveal:

- Full names
- Job titles
- Reporting structures

Tools such as `linkedin2username` can automate employee-name collection and generate username lists.

---

## GitHub and GitLab

Developers may commit code using their corporate email addresses.

This can reveal:

- Organisation email format
- Individual usernames

---

## Public Data Breaches

Breach databases may contain email addresses from the target domain.

These can directly reveal the organisation's username format.

---

## Corporate Websites

Pages such as:

- About Us
- Meet the Team

may list employee names that can be converted into potential usernames.

---

## Job Listings

Recruitment posts can reveal:

- Internal technologies
- Team structures
- Naming conventions

---

# 🧪 Lab Note

In the TryHackMe lab, real OSINT is not performed.

Instead, a wordlist of potential usernames is provided.

The methodology is still important because real engagements require understanding where username lists originate.

---

# 👤 Common Username Formats

For **Jane Smith**:

| Format | Example |
|---|---|
| `first.last` | `jane.smith` |
| `firstlast` | `janesmith` |
| `flast` | `jsmith` |
| `first.l` | `jane.s` |
| `first` | `jane` |
| `last.first` | `smith.jane` |

The key objective is to identify the naming convention used by the organisation.

A single confirmed email address or username can sometimes reveal the convention.

---

# 🔐 Username Enumeration with Kerbrute

Kerbrute can validate potential usernames against the target domain using Kerberos pre-authentication behaviour.

When an AS-REQ is sent:

### Non-existent User

The KDC returns:

```text
KDC_ERR_C_PRINCIPAL_UNKNOWN
```

### Existing User

The KDC requests pre-authentication.

This confirms that the account exists.

---

# ⚠️ Detection Consideration

Kerberos username enumeration does not trigger normal account lockouts because failed pre-authentication requests are not counted as failed login attempts.

However, it is **not completely silent**.

It generates:

```text
Windows Event ID 4768
Kerberos Authentication Service Requests
```

on the Domain Controller.

---

# 💻 Enumerating Usernames

Place the provided username wordlist at:

```text
/root/usernames.txt
```

Then run:

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt
```

---

# 🔍 Command Breakdown

```text
userenum
```

Kerbrute's username enumeration module.

```text
-d thm.loc
```

Target domain / Kerberos realm.

```text
--dc 192.168.12.100
```

Domain Controller IP.

```text
/root/usernames.txt
```

Potential username wordlist.

---

# 📄 Example Output

```text
[+] VALID USERNAME: jane.smith@thm.loc
[+] VALID USERNAME: bob.taylor@thm.loc
[+] VALID USERNAME: admin.svc@thm.loc
```

Each:

```text
[+] VALID USERNAME
```

represents a confirmed domain account.

These usernames can be used for subsequent password spraying.

---

# 💾 Save Kerbrute Results

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt -o valid_users.txt
```

---

# 🌐 DNS Enumeration

DNS is the backbone of Active Directory.

Domain Controllers, mail servers and other critical infrastructure are registered through DNS.

---

## Identify Domain Controllers

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.thm.loc 192.168.12.100
```

---

## Identify the Kerberos KDC

```bash
nslookup -type=SRV _kerberos._tcp.thm.loc 192.168.12.100
```

---

## Identify Mail Servers

```bash
nslookup -type=MX thm.loc 192.168.12.100
```

---

# 🧠 Why DNS Enumeration Matters

These queries help map:

- Domain Controllers
- Kerberos infrastructure
- Mail servers
- Network topology

Knowing where critical services sit provides a clearer picture before active attacks begin.

---

# 💡 Key Takeaways

- OSINT is used to build potential username lists.
- LinkedIn, GitHub/GitLab, public breaches, corporate websites and job listings can provide useful information.
- Organisations often use predictable username formats.
- Kerbrute can validate usernames using Kerberos behaviour.
- Valid usernames produce different KDC responses from invalid usernames.
- Kerbrute username enumeration does not normally trigger account lockouts.
- Event ID 4768 can reveal Kerberos authentication-service requests.
- DNS enumeration helps identify important AD infrastructure.
