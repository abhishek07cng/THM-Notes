# 🏢 Organizational Units (OUs) and Active Directory Management

> **Module:** Active Directory Basics
>
> Organizational Units (OUs) are logical containers in Active Directory used to organize users, computers, and other objects. They simplify administration and allow administrators to apply different security policies to different parts of an organization.

---

# 📖 Overview

As organizations grow, managing hundreds or thousands of users and computers becomes increasingly difficult.

To simplify administration, Active Directory organizes objects into **Organizational Units (OUs)**.

Administrators use the **Active Directory Users and Computers (ADUC)** management console to create, modify, and organize these objects. :contentReference[oaicite:1]{index=1}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- What Active Directory Users and Computers (ADUC) is
- What Organizational Units (OUs) are
- Why OUs are important
- Default containers in Active Directory
- Difference between OUs and Security Groups

---

# 🖥 Active Directory Users and Computers (ADUC)

**Active Directory Users and Computers (ADUC)** is Microsoft's graphical management console used to administer Active Directory.

Using ADUC, administrators can:

- Create users
- Delete users
- Modify user accounts
- Create groups
- Manage computers
- Reset passwords
- Organize Active Directory objects

It provides a centralized interface for managing all domain objects. :contentReference[oaicite:2]{index=2}

---

# 📂 What are Organizational Units (OUs)?

An **Organizational Unit (OU)** is a container object within Active Directory.

Its primary purpose is to organize users, computers, and other objects into logical groups.

For example:

```
THM

├── IT

├── Management

├── Marketing

├── R&D

└── Sales
```

This structure reflects the departments within an organization and makes administration much easier. :contentReference[oaicite:3]{index=3}

---

# 🎯 Why Use Organizational Units?

Organizational Units allow administrators to:

- Organize Active Directory objects logically.
- Apply different Group Policies to different departments.
- Delegate administrative responsibilities.
- Simplify large enterprise environments.

For example:

- IT department → Administrative privileges
- Sales department → Restricted permissions
- Marketing department → Different desktop policies

Each department can receive policies tailored to its specific requirements. :contentReference[oaicite:4]{index=4}

---

# 👤 User Membership in OUs

An important limitation highlighted in the TryHackMe room is:

> A user can belong to **only one Organizational Unit (OU) at a time**.

This avoids conflicts that could arise if multiple sets of organizational policies were applied simultaneously. :contentReference[oaicite:5]{index=5}

---

# 🔧 Managing Objects in ADUC

Administrators can perform several common tasks directly within ADUC, including:

- Creating users
- Deleting users
- Editing user properties
- Resetting passwords
- Managing computers
- Managing groups

Password resets are especially useful for helpdesk and IT support teams. :contentReference[oaicite:6]{index=6}

---

# 📦 Default Containers in Active Directory

Windows automatically creates several containers during Active Directory installation.

| Container | Purpose |
|-----------|---------|
| **Builtin** | Contains default Windows groups available on every domain |
| **Computers** | Default location for newly joined computers |
| **Domain Controllers** | Contains all Domain Controllers |
| **Users** | Default users and groups |
| **Managed Service Accounts** | Stores service accounts used by applications | :contentReference[oaicite:7]{index=7}

---

# 🔄 Security Groups vs Organizational Units

Although both are used to organize objects, they serve different purposes.

## Organizational Units (OUs)

Used for:

- Organizing objects
- Applying Group Policies
- Administrative delegation

Characteristics:

- One user belongs to one OU.
- Policies are applied to the entire OU.

---

## Security Groups

Used for:

- Granting permissions
- Controlling access to resources

Characteristics:

- Users can belong to multiple Security Groups.
- Permissions are inherited from all assigned groups.

Examples:

- Access to shared folders
- Network printers
- Applications
- File permissions

The TryHackMe room emphasizes that OUs manage **policies**, while Security Groups manage **permissions**. :contentReference[oaicite:8]{index=8}

---

# 📊 OU vs Security Group

| Organizational Unit | Security Group |
|---------------------|----------------|
| Organizes objects | Grants permissions |
| Applies Group Policies | Controls resource access |
| One OU per user | Multiple groups per user |
| Administrative organization | Permission management |

---

# 🛠 Tools Used

## Active Directory Users and Computers (ADUC)

### Purpose

Graphical management console for Active Directory.

### Common Tasks

- Create users
- Delete users
- Reset passwords
- Manage groups
- Manage computers
- Create Organizational Units

---

# 💡 Key Takeaways

- ADUC is the primary graphical tool for managing Active Directory.
- Organizational Units help organize domain objects logically.
- OUs simplify policy management.
- Users belong to only one OU.
- Security Groups control permissions, not policies.
- Users can belong to multiple Security Groups.
- Windows creates several default containers automatically.

---

# 📚 References

- TryHackMe – Active Directory Basics