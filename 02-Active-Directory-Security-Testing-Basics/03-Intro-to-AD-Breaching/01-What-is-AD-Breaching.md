# 🔓 What Is AD Breaching?

> **Topic:** Introduction to AD Breaching
>
> **Section:** What is AD Breaching?

---

# 📖 Overview

**AD breaching** is the process of obtaining an initial set of valid **Active Directory credentials** when starting from scratch.

It is the first phase of an Active Directory attack chain.

```text
Initial Access
      ↓
Valid AD Credentials
      ↓
Domain Enumeration
      ↓
Lateral Movement
      ↓
Privilege Escalation
```

Without that first set of credentials, an attacker has limited ability to enumerate the domain, move laterally, or escalate privileges.

---

# 🔑 Why Initial Credentials Matter

Even a low-privileged domain account can expose significant information.

Once authenticated, a user can query Active Directory for information such as:

- Users
- Groups
- Computers
- Group Policies
- Trust relationships

This information can reveal:

```text
Misconfigurations
      ↓
Attack Paths
      ↓
Privilege Escalation
      ↓
Domain Admin
```

The hardest part of an AD engagement can therefore be obtaining the first foothold.

---

# 🌐 AD Attack Surface

A typical AD environment exposes several services and protocols that can be targeted during the breaching phase.

| Service / Protocol | Port | Potential Relevance |
|---|---:|---|
| **SMB** | TCP 445 | File shares, printers, remote administration, password spraying and credential testing |
| **LDAP** | TCP 389 / 636 | Directory access and possible credential exposure |
| **HTTP / HTTPS** | 80 / 443 | Internal portals, CI/CD platforms and device management interfaces |
| **Kerberos** | TCP/UDP 88 | Authentication and username enumeration |
| **DNS** | TCP/UDP 53 | Identifying Domain Controllers, mail servers and other infrastructure |

Each service can provide an avenue for obtaining the first set of credentials through:

- Credential discovery
- Password spraying
- Coercion attacks

---

# 🚩 Starting Positions

In a real engagement, you will generally start from one of two positions.

## Unauthenticated — Black Box

You have:

- Network access
- No valid domain credentials

The objective is to:

```text
Enumerate
   ↓
Spray
   ↓
Coerce
   ↓
Obtain Valid Credentials
```

---

## Authenticated — Grey Box

You already have a valid set of low-privileged credentials.

Possible sources include:

- Earlier engagement activity
- Phishing
- OSINT discovery

From here, you can move directly into:

```text
Enumeration
     ↓
Credential Discovery
     ↓
Privilege Escalation
     ↓
Lateral Movement
```

---

# 🎯 Goal

Regardless of the starting position:

> **The goal is to obtain valid AD credentials that allow us to move deeper into the environment.**

The following sections cover several common techniques for achieving that initial foothold.

---

# 💡 Key Takeaways

- AD breaching is the process of obtaining initial valid AD credentials.
- It represents the first phase of an AD attack chain.
- Low-privileged credentials can expose valuable domain information.
- SMB, LDAP, HTTP/HTTPS, Kerberos and DNS are important parts of the AD attack surface.
- An engagement may begin from an unauthenticated or authenticated position.
- The objective is to obtain valid credentials and move deeper into the environment.
