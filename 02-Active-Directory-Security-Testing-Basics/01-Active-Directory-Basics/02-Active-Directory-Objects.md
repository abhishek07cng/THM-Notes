# 🧩 Active Directory Objects

> **Module:** Active Directory Basics
>
> Active Directory stores and manages different types of objects that represent users, computers, groups, and other resources within a Windows Domain.

---

# 📖 Overview

The core component of a Windows Domain is **Active Directory Domain Services (AD DS)**.

AD DS acts as a centralized directory that stores information about every object within the network.

Examples of Active Directory objects include:

- Users
- Computers (Machines)
- Security Groups
- Printers
- Shared Folders
- Services

These objects allow administrators to centrally manage identities and resources across the domain. :contentReference[oaicite:1]{index=1}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- What AD DS is
- What Active Directory Objects are
- Different types of Users
- Machine Accounts
- Security Principals
- Security Groups
- Default Domain Groups

---

# 📚 Active Directory Domain Services (AD DS)

Active Directory Domain Services (AD DS) is the core service of Active Directory.

Its primary responsibility is to maintain a centralized catalogue containing information about every object in the domain.

Examples include:

- Users
- Groups
- Machines
- Printers
- Shared Resources

Instead of storing information separately on every computer, AD DS stores it centrally for easier administration. :contentReference[oaicite:2]{index=2}

---

# 👤 Users

Users are one of the most common object types in Active Directory.

Users are considered **Security Principals**, meaning they can:

- Authenticate to the domain
- Access network resources
- Receive permissions
- Be assigned privileges

A **Security Principal** is any object that can be authenticated and authorized to interact with resources on the network. :contentReference[oaicite:3]{index=3}

---

# 👥 Types of User Accounts

The TryHackMe room describes two common types of user accounts.

## 1. People Accounts

These represent actual users within an organization.

Examples:

- Employees
- Administrators
- Students
- Teachers

These accounts allow individuals to access domain resources.

---

## 2. Service Accounts

Some applications require their own dedicated user account to run.

Examples include services such as:

- IIS
- Microsoft SQL Server (MSSQL)

Service accounts are configured with only the permissions necessary for the specific service they operate. :contentReference[oaicite:4]{index=4}

---

# 💻 Machine Accounts

Whenever a computer joins an Active Directory domain, a **Machine Account** is automatically created.

Machine accounts are also **Security Principals** and receive their own domain account.

These accounts:

- Authenticate to the domain
- Have limited permissions
- Are primarily used by the computer itself

Normally, users should never interact directly with machine accounts. :contentReference[oaicite:5]{index=5}

---

# 🔑 Machine Account Passwords

The TryHackMe room notes that:

- Machine account passwords are automatically rotated.
- They are generally composed of **120 random characters**.

This reduces the likelihood of compromise through weak or reused passwords. :contentReference[oaicite:6]{index=6}

---

# 🏷 Naming Convention

Machine accounts follow a standard naming convention:

```
Computer Name + $
```

Example:

```
DC01
```

Machine Account:

```
DC01$
```

The trailing **`$`** identifies the account as a machine account. :contentReference[oaicite:7]{index=7}

---

# 👥 Security Groups

Managing permissions user-by-user quickly becomes difficult in large environments.

Instead, Active Directory uses **Security Groups**.

A Security Group:

- Contains users
- Can contain computers
- Can contain other groups
- Is itself a Security Principal

Permissions are assigned to the group rather than to individual users.

Users inherit the permissions of every group they belong to. :contentReference[oaicite:8]{index=8}

---

# 🏢 Default Security Groups

Active Directory creates several important groups by default.

| Security Group | Purpose |
|---------------|---------|
| Domain Admins | Full administrative control over the domain |
| Server Operators | Manage Domain Controllers (except administrative group membership) |
| Backup Operators | Access all files for backup purposes |
| Account Operators | Create and modify domain accounts |
| Domain Users | Contains all user accounts |
| Domain Computers | Contains all domain-joined computers |
| Domain Controllers | Contains all Domain Controllers | :contentReference[oaicite:9]{index=9}

---

# 🔄 Relationship Between Objects

```text
Active Directory

│

├── Users

├── Machine Accounts

├── Security Groups

├── Printers

├── Shared Resources

└── Other Objects
```

---

# 🛠 Tools Used

No graphical administration tools are introduced in this section.

The concepts discussed here form the foundation for managing Active Directory objects in later sections.

---

# 💡 Key Takeaways

- AD DS stores information about all objects in the domain.
- Users and Machine Accounts are both Security Principals.
- Service Accounts allow applications to run securely.
- Machine accounts automatically receive domain accounts.
- Machine account names end with a **`$`**.
- Security Groups simplify permission management.
- Default groups provide predefined administrative roles.

---

# 📚 References

- TryHackMe – Active Directory Basics