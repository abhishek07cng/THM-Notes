# 🔐 Kerbrute Username Enumeration

> **Topic:** AD Basic Enumeration
>
> **Section:** Username Enumeration with Kerbrute

---

# 📖 Overview

Kerberos is the primary authentication protocol used by Microsoft Windows domains.

Unlike NTLM's challenge-response mechanism, Kerberos uses a ticket-based system managed by a trusted third party called the:

```text
Key Distribution Centre (KDC)
```

Kerbrute can enumerate valid AD users by interacting with Kerberos pre-authentication.

---

# 🧩 Why Validate Usernames?

Tools such as:

- enum4linux-ng
- rpcclient

may return usernames that are:

- Disabled accounts
- Non-domain accounts
- Fake honeypot users
- False positives

Kerbrute can help confirm which accounts are valid.

```text
Collected Usernames
        ↓
Kerbrute
        ↓
Valid AD Users
        ↓
Password Spraying
```

---

# 📄 Creating a User List

Example:

```bash
cat users.txt
```

```text
Administrator
Guest
krbtgt
sshd
gerald.burgess
nigel.parsons
...
asrepuser1
rduke
```

---

# 🛠️ Kerbrute Installation

The supplied material describes the following installation process.

## 1. Download the Precompiled Binary

Download a suitable binary for the operating system from the Kerbrute releases page.

```text
https://github.com/ropnop/kerbrute/releases
```

## 2. Rename the Binary

Example:

```text
kerbrute_linux_amd64
```

to:

```text
kerbrute
```

## 3. Make It Executable

```bash
chmod +x kerbrute
```

> **Lab Note:** The source states that Kerbrute is not installed on the AttackBox and requires internet access to download.

---

# 🔎 Username Enumeration

Run:

```bash
./kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users.txt
```

---

# 🔍 Command Breakdown

```text
userenum
```

Performs username enumeration.

```text
--dc 10.211.11.10
```

Specifies the Domain Controller.

```text
-d tryhackme.loc
```

Specifies the AD domain.

```text
users.txt
```

Provides the candidate username list.

---

# 📄 Example Output

```text
[+] VALID USERNAME: WRK$@tryhackme.loc
[+] VALID USERNAME: guy.smith@tryhackme.loc
[+] VALID USERNAME: sshd@tryhackme.loc
[+] VALID USERNAME: nigel.parsons@tryhackme.loc
[+] VALID USERNAME: gerald.burgess@tryhackme.loc
[+] VALID USERNAME: administrator@tryhackme.loc
```

The source's lab run tested:

```text
33 usernames
```

and identified:

```text
28 valid usernames
```

---

# 🧠 Why Kerbrute Matters

If other enumeration methods are unavailable, Kerbrute can be used with a username wordlist to discover valid accounts.

The resulting list can then be used for:

```text
Password Spraying
```

---

# 🔄 Enumeration Workflow

```text
Host Discovery
      ↓
Domain Controller
      ↓
SMB / LDAP / RPC
      ↓
Candidate Usernames
      ↓
Kerbrute
      ↓
Valid Users
      ↓
Password Spraying
```

---

# 💡 Key Takeaways

- Kerbrute enumerates users through Kerberos.
- It helps validate usernames gathered through SMB, LDAP and RPC enumeration.
- Some usernames returned by other tools may be disabled or invalid.
- A clean valid-user list improves the accuracy of subsequent password spraying.
- The source demonstrates `userenum` with a specified Domain Controller and domain.
