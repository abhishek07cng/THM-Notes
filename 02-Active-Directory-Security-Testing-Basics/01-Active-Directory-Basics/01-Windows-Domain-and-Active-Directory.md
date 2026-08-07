# 🏢 Windows Domain and Active Directory

> **Module:** Active Directory Basics
>
> This note introduces the concept of Windows Domains, explains why they are used in enterprise environments, and describes the role of Active Directory (AD) and Domain Controllers (DCs).

---

# 📖 Overview

Managing a small network is relatively simple. For example, if a company has only a few computers and users, an administrator can manually configure each system, create local user accounts, and manage settings individually.

However, as an organization grows to hundreds or even thousands of computers across multiple offices, managing each machine independently becomes inefficient and nearly impossible.

To solve this problem, Microsoft introduced **Windows Domains**, which centralize the management of users, computers, and security policies through **Active Directory (AD)**. :contentReference[oaicite:0]{index=0}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- What a Windows Domain is
- Why organizations use Windows Domains
- What Active Directory (AD) is
- What a Domain Controller (DC) does
- Advantages of centralized administration
- A real-world example of Active Directory

---

# 🌐 What is a Windows Domain?

A **Windows Domain** is a collection of users, computers, and other network resources managed centrally by an organization.

Instead of configuring every computer individually, administrators manage the entire environment from a central location using **Active Directory**. :contentReference[oaicite:1]{index=1}

### Without a Windows Domain

- Each computer has its own local users.
- Passwords must be managed separately.
- Security settings are configured on every machine.
- Administration becomes difficult as the network grows.

### With a Windows Domain

- User accounts are managed centrally.
- Authentication is centralized.
- Security policies are deployed across the organization.
- Users can access multiple domain-joined computers with the same credentials.

---

# 📂 What is Active Directory?

**Active Directory (AD)** is Microsoft's directory service used to manage and organize network resources within a Windows Domain.

It acts as a **central repository** that stores information about:

- Users
- Computers
- Groups
- Printers
- Shared folders
- Policies
- Other network objects

Rather than storing this information separately on every computer, Active Directory maintains it in one centralized location. :contentReference[oaicite:2]{index=2}

---

# 🖥️ What is a Domain Controller (DC)?

A **Domain Controller (DC)** is a Windows Server that runs **Active Directory Domain Services (AD DS)**.

The Domain Controller is responsible for:

- Authenticating users
- Storing Active Directory data
- Applying security policies
- Managing domain resources
- Handling authentication requests from users and computers

Whenever a user signs in with domain credentials, the authentication request is forwarded to the Domain Controller for verification. :contentReference[oaicite:3]{index=3}

---

# ✅ Advantages of a Windows Domain

The TryHackMe room highlights two major advantages:

### 1. Centralized Identity Management

Administrators can create, modify, or remove user accounts from Active Directory instead of managing every computer individually.

### 2. Centralized Security Policies

Security settings can be configured once and applied across all users and computers in the domain through Group Policy. :contentReference[oaicite:4]{index=4}

---

# 🌍 Real-World Example

Many schools, universities, and workplaces use Active Directory.

For example:

- A student receives one username and password.
- They can log in to any domain-joined computer on campus.
- The computer sends the authentication request to Active Directory.
- If the credentials are valid, access is granted.

Because authentication is centralized, the same credentials work across multiple machines without creating separate local accounts. :contentReference[oaicite:5]{index=5}

---

# 🔒 Centralized Policy Management

Active Directory also allows organizations to enforce security restrictions.

Examples include:

- Preventing users from opening the Control Panel.
- Restricting administrative privileges.
- Applying password policies.
- Configuring security settings for all computers.

These policies are managed centrally and applied consistently throughout the domain. :contentReference[oaicite:6]{index=6}

---

# 🔄 Windows Domain Workflow

```text
User

↓

Domain-Joined Computer

↓

Authentication Request

↓

Domain Controller (Active Directory)

↓

Credentials Verified

↓

Access Granted / Denied
```

---

# 🛠 Tools Used

No administrative tools are introduced in this section.

The following tools will be covered later in this module:

- Active Directory Users and Computers (ADUC)
- Group Policy Management (GPMC)

---

# 💡 Key Takeaways

- A Windows Domain centralizes the management of users, computers, and security.
- Active Directory is the core directory service used within a Windows Domain.
- Domain Controllers host Active Directory Domain Services (AD DS).
- Centralized authentication allows users to log in across multiple domain-joined computers.
- Organizations use Active Directory to simplify administration and enforce consistent security policies.

---

# 📚 References

- TryHackMe – Active Directory Basics