# 📜 Group Policy Objects (GPO)

> **Module:** Active Directory Basics
>
> Group Policy Objects (GPOs) allow administrators to centrally configure and enforce security settings, user configurations, and computer policies across an Active Directory environment.

---

# 📖 Overview

Organizing users and computers into Organizational Units (OUs) is only the first step.

The real power of Active Directory comes from **Group Policy Objects (GPOs)**, which allow administrators to automatically configure hundreds of Windows settings across an entire organization.

Instead of manually configuring every computer, administrators define a policy once and deploy it to many users or computers simultaneously. :contentReference[oaicite:1]{index=1}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- What a Group Policy Object (GPO) is
- How GPOs work
- Group Policy Management Console (GPMC)
- GPO Linking
- Security Filtering
- Password Policies
- SYSVOL
- Group Policy Updates
- Practical GPO Examples

---

# 📚 What is a Group Policy Object (GPO)?

A **Group Policy Object (GPO)** is a collection of configuration settings that can be applied to users or computers within Active Directory.

A GPO can configure:

- Password policies
- Account lockout policies
- Desktop settings
- Control Panel restrictions
- Screen lock policies
- Windows security settings
- Software deployment
- Hundreds of other Windows configurations

Rather than configuring each computer individually, administrators configure the GPO once and apply it across the required Organizational Units (OUs). :contentReference[oaicite:2]{index=2}

---

# 🖥 Group Policy Management Console (GPMC)

Administrators manage GPOs using the **Group Policy Management Console (GPMC)**.

Typical workflow:

```text
Create GPO

↓

Configure Policies

↓

Link GPO

↓

Clients Receive Policy

↓

Windows Applies Configuration
```

The GPMC displays:

- Domain hierarchy
- Organizational Units
- Existing GPOs
- Linked policies
- Security filtering

:contentReference[oaicite:3]{index=3}

---

# 🔗 Linking a GPO

A GPO does not affect users or computers until it is **linked**.

A GPO can be linked to:

- Entire Domain
- Organizational Unit (OU)

Once linked, the GPO applies to:

- The linked OU
- All child OUs beneath it

Example:

```text
thm.local

↓

Marketing

↓

Sales
```

If a GPO is linked to **thm.local**, both **Marketing** and **Sales** inherit the policy unless blocked. :contentReference[oaicite:4]{index=4}

---

# 🎯 Security Filtering

By default, a GPO applies to the **Authenticated Users** group.

Administrators can use **Security Filtering** to limit which users or computers receive the policy.

This allows more precise control without changing the OU structure. :contentReference[oaicite:5]{index=5}

---

# ⚙ Computer vs User Configuration

Every GPO contains two major sections:

## Computer Configuration

Applies to:

- Computers
- Servers
- Domain Controllers

Examples:

- Password Policy
- Windows Firewall
- Security Settings

---

## User Configuration

Applies to:

- User Accounts

Examples:

- Desktop Restrictions
- Control Panel
- Start Menu
- User Experience

:contentReference[oaicite:6]{index=6}

---

# 🔐 Password Policy Example

The TryHackMe room demonstrates modifying the **Default Domain Policy**.

Example:

```
Minimum Password Length

↓

10 Characters
```

This policy is configured through:

```text
Computer Configuration

↓

Policies

↓

Windows Settings

↓

Security Settings

↓

Account Policies

↓

Password Policy
```

Since the Default Domain Policy is linked to the domain, changing it affects all domain users. :contentReference[oaicite:7]{index=7}

---

# 📂 SYSVOL

Group Policies are distributed through the **SYSVOL** network share.

Default location:

```text
C:\Windows\SYSVOL\sysvol\
```

Every domain computer periodically synchronizes Group Policies from SYSVOL.

This ensures policy consistency across the environment. :contentReference[oaicite:8]{index=8}

---

# 🔄 Updating Group Policies

Normally, clients refresh Group Policies automatically.

To force an immediate update:

```powershell
gpupdate /force
```

This downloads and applies the latest policies without waiting for the normal refresh interval. :contentReference[oaicite:9]{index=9}

---

# 🧪 Practical Example 1 – Restrict Control Panel

The TryHackMe room creates a GPO named:

```
Restrict Control Panel Access
```

Goal:

Prevent users in:

- Marketing
- Management
- Sales

from opening the Control Panel.

This policy is configured under **User Configuration** and linked only to the required departmental OUs. :contentReference[oaicite:10]{index=10}

---

# 🧪 Practical Example 2 – Auto Lock Screen

The room also creates another GPO:

```
Auto Lock Screen
```

Goal:

Automatically lock computers after:

```
5 Minutes
```

of inactivity.

Because every workstation, server, and Domain Controller should receive this setting, the GPO is linked to the **root domain**, allowing child OUs to inherit it. :contentReference[oaicite:11]{index=11}

---

# 📊 GPO Workflow

```text
Administrator

↓

Create GPO

↓

Configure Settings

↓

Link to Domain / OU

↓

SYSVOL Replication

↓

Client Refresh

↓

Policy Applied
```

---

# 💡 Best Practices

- Keep GPOs focused on a specific purpose.
- Link GPOs only where required.
- Use Security Filtering when necessary.
- Test policies before production deployment.
- Organize computers into appropriate OUs before applying policies.

---

# 📝 TryHackMe Summary

The TryHackMe room demonstrates how GPOs simplify centralized administration by allowing administrators to deploy password policies, restrict Control Panel access, and enforce automatic screen locking. Policies are distributed through **SYSVOL** and can be refreshed immediately using `gpupdate /force`. :contentReference[oaicite:12]{index=12}

---

# 💡 Key Takeaways

- GPOs centralize Windows configuration management.
- GPMC is used to create and manage GPOs.
- GPOs must be linked before they take effect.
- Child OUs inherit linked policies.
- SYSVOL distributes Group Policies.
- `gpupdate /force` forces immediate policy synchronization.

---

# 📚 References

- TryHackMe – Active Directory Basics