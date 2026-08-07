# 👥 Managing Users and Computers in Active Directory

> **Module:** Active Directory Basics
>
> Proper organization of users and computers is essential for maintaining a secure, scalable, and manageable Active Directory environment.

---

# 📖 Overview

By default, every computer that joins an Active Directory domain is placed inside the **Computers** container.

Although this works for small environments, it quickly becomes difficult to manage as the organization grows.

To improve administration, computers should be organized into dedicated Organizational Units (OUs) based on their purpose. :contentReference[oaicite:0]{index=0}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- Default Computers container
- Why computer organization matters
- Types of computers in Active Directory
- Best practices for organizing devices
- Creating dedicated Organizational Units

---

# 📂 Default Computers Container

Whenever a machine joins the domain, it is automatically placed in the **Computers** container.

Examples include:

- Desktop computers
- Employee laptops
- Application servers

While functional, storing every device in one location makes policy management difficult. :contentReference[oaicite:1]{index=1}

---

# 🖥 Types of Devices

The TryHackMe room groups devices into three main categories.

---

## 1. Workstations

Workstations are the computers used daily by employees.

Examples:

- Office desktops
- Employee laptops

Characteristics:

- Used for everyday work
- Used for web browsing
- Should **not** have privileged users permanently logged in

:contentReference[oaicite:2]{index=2}

---

## 2. Servers

Servers provide services to users and other systems.

Examples:

- File Servers
- Web Servers
- Database Servers
- DNS Servers

Servers typically remain online continuously and provide shared resources throughout the organization. :contentReference[oaicite:3]{index=3}

---

## 3. Domain Controllers

Domain Controllers are the most important systems within an Active Directory environment.

Responsibilities include:

- Storing Active Directory data
- Authenticating users
- Managing Group Policies
- Managing domain resources

Because they contain sensitive information such as password hashes, Domain Controllers are considered the most critical systems in the environment. :contentReference[oaicite:4]{index=4}

---

# 🏗 Organizing Devices

Instead of leaving every machine inside the **Computers** container, administrators should organize them into dedicated OUs.

Example structure:

```text
thm.local

├── Domain Controllers

├── Servers

└── Workstations
```

This separation makes administration easier and prepares the environment for applying different Group Policies. :contentReference[oaicite:5]{index=5}

---

# 🎯 Why Organize Devices?

Separating computers into dedicated OUs allows administrators to:

- Apply different security policies
- Simplify administration
- Delegate management
- Reduce configuration errors
- Improve scalability

---

# 📊 Recommended Organization

| OU | Typical Devices |
|----|-----------------|
| Workstations | Employee desktops and laptops |
| Servers | Application and infrastructure servers |
| Domain Controllers | Domain Controllers only |

---

# 🛠 Administrative Workflow

```text
Computer Joins Domain

↓

Computers Container

↓

Administrator Reviews Device

↓

Move Device

↓

Workstations / Servers / Domain Controllers OU

↓

Apply Appropriate Policies
```

---

# 💡 Best Practices

- Separate workstations from servers.
- Never use Domain Controllers for daily work.
- Keep privileged accounts away from workstations.
- Organize devices before deploying Group Policies.
- Use meaningful OU structures.

---

# 📝 TryHackMe Summary

The TryHackMe room recommends moving newly joined devices out of the default **Computers** container and organizing them into dedicated OUs such as **Workstations** and **Servers**. This makes future policy deployment and administration significantly easier. :contentReference[oaicite:6]{index=6}

---

# 💡 Key Takeaways

- Computers join the **Computers** container by default.
- Devices should be organized according to their role.
- Domain Controllers are the most sensitive systems.
- Proper organization simplifies administration and policy management.

---

# 📚 References

- TryHackMe – Active Directory Basics