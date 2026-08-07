# 🌳 Trees, Forests, and Trusts

> **Module:** Active Directory Basics
>
> As organizations grow, a single Active Directory domain may no longer be sufficient. Active Directory supports multiple domains through **Trees**, **Forests**, and **Trust Relationships**, enabling centralized management across large enterprise environments.

---

# 📖 Overview

Large organizations often consist of multiple departments, subsidiaries, or geographic locations that require separate Active Directory domains.

Instead of creating isolated environments, Active Directory allows multiple domains to communicate securely using **Trees**, **Forests**, and **Trust Relationships**. These structures provide centralized administration while allowing each domain to maintain its own policies and resources. :contentReference[oaicite:1]{index=1}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- What an Active Directory Tree is
- What an Active Directory Forest is
- Parent and Child Domains
- Enterprise Admins
- Trust Relationships
- One-way and Two-way Trusts
- Why Trusts are required

---

# 🌳 Active Directory Trees

An **Active Directory Tree** is a collection of one or more domains that share:

- A common namespace
- A hierarchical relationship
- Automatic trust relationships

A child domain inherits part of the parent's DNS namespace.

Example:

```text
thm.local
│
├── research.thm.local
│
├── sales.thm.local
│
└── hr.thm.local
```

Each child domain maintains its own users and resources while remaining part of the same tree. :contentReference[oaicite:2]{index=2}

---

# 🌲 Active Directory Forests

A **Forest** is the highest-level logical structure in Active Directory.

Unlike a tree, a forest can contain multiple trees with different DNS namespaces.

Example:

```text
thm.local

↓

Tree 1

company.local

↓

Tree 2

research.org
```

Although the namespaces differ, the trees belong to the same forest and can share resources through trust relationships. :contentReference[oaicite:3]{index=3}

---

# 👑 Enterprise Admins

Within a forest, the **Enterprise Admins** group has administrative privileges across every domain.

Members of this group can:

- Manage all domains
- Create new domains
- Modify forest-wide settings
- Perform enterprise-level administration

Because of these extensive privileges, Enterprise Admin accounts should be carefully protected. :contentReference[oaicite:4]{index=4}

---

# 🤝 Trust Relationships

A **Trust Relationship** allows users from one domain to access resources in another domain.

Without trusts, domains operate independently and cannot authenticate users from one another.

Trusts enable:

- Resource sharing
- Cross-domain authentication
- Centralized collaboration

This is especially useful in large organizations with multiple domains. :contentReference[oaicite:5]{index=5}

---

# 🔄 Types of Trusts

## One-Way Trust

In a one-way trust:

- Domain A trusts Domain B
- Users in Domain B can access resources in Domain A (subject to permissions)
- The reverse is **not** automatically true

```text
Domain A  ←  Domain B
```

---

## Two-Way Trust

In a two-way trust:

- Domain A trusts Domain B
- Domain B trusts Domain A

Both domains can authenticate users from the other domain, provided appropriate permissions are granted.

```text
Domain A  ⇄  Domain B
```

The TryHackMe room highlights these two trust models as the primary methods for establishing communication between domains. :contentReference[oaicite:6]{index=6}

---

# 🏗 Active Directory Hierarchy

```text
Forest
│
├── Tree
│   ├── Parent Domain
│   │
│   ├── Child Domain
│   │
│   └── Child Domain
│
└── Tree
    ├── Parent Domain
    │
    └── Child Domain
```

---

# 📊 Trees vs Forests

| Tree | Forest |
|------|--------|
| Collection of related domains | Collection of one or more trees |
| Shared DNS namespace | May contain different DNS namespaces |
| Automatic trust between parent and child domains | Trusts exist across the forest |
| Smaller logical structure | Highest-level Active Directory structure |

---

# 📝 TryHackMe Summary

As organizations expand, Active Directory can be structured into **Trees** and **Forests** to support multiple domains. Trust relationships allow users to authenticate and access resources across domains while maintaining centralized administration. Enterprise Admins have administrative privileges across the entire forest. :contentReference[oaicite:7]{index=7}

---

# 💡 Key Takeaways

- A Tree is a hierarchy of related domains sharing a namespace.
- A Forest is the highest-level Active Directory structure.
- Enterprise Admins manage the entire forest.
- Trusts allow domains to share resources securely.
- Trusts can be one-way or two-way depending on organizational requirements.

---

# 📚 References

- TryHackMe – Active Directory Basics