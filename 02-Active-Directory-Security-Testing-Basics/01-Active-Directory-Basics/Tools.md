# 🛠 Tools Used — Active Directory Basics

This document lists the tools and utilities introduced in the **Active Directory Basics** module.

---

# 1. Active Directory Users and Computers (ADUC)

## Purpose

Microsoft Management Console (MMC) snap-in used to manage Active Directory objects.

### Common Tasks

- Create users
- Delete users
- Reset passwords
- Create groups
- Create Organizational Units
- Manage computers

### Used In

- Organizational Units and AD Management

---

# 2. Group Policy Management Console (GPMC)

## Purpose

Used to create, edit, manage, and deploy Group Policy Objects (GPOs).

### Common Tasks

- Create GPOs
- Edit GPOs
- Link GPOs
- Security Filtering
- View Group Policy inheritance

### Used In

- Group Policy Objects (GPO)

---

# 3. Windows PowerShell

## Purpose

Windows command-line shell and scripting environment used for administration and automation.

### Example Usage

```powershell
gpupdate /force
```

### Used In

- Group Policy refresh

---

# 4. gpupdate

## Purpose

Refreshes Group Policy settings immediately without waiting for the automatic refresh interval.

### Syntax

```powershell
gpupdate /force
```

### Used In

- Applying newly created Group Policies

---

# 📌 Summary

| Tool | Purpose |
|------|---------|
| ADUC | Manage Active Directory objects |
| GPMC | Manage Group Policy Objects |
| Windows PowerShell | Windows administration and automation |
| gpupdate | Force Group Policy refresh |