# 🔐 Authentication in Active Directory

> **TryHackMe Room:** Intro to AD Authentication
>
> **Task:** 2 — Authentication in AD

---

# 📖 Overview

When you authenticate, you provide some form of credential that proves your identity.

In Active Directory environments, the most common form of authentication material is a **username and password**, but other forms of authentication material can also be used.

The goal remains the same regardless of the method:

> **Prove to the domain that you are who you claim to be.**

---

# 🔑 Authentication Material

Authentication material can take several forms.

## 1. Username and Password

The most common authentication method.

A user provides:

```text
Username
Password
```

The password represents:

> **Something you know.**

---

## 2. Certificates

A cryptographic certificate issued by a trusted **Certificate Authority (CA)** can be used to prove identity.

Certificates are often used for:

- Machine authentication
- Smart card logins

### Authentication Flow

```text
User / Machine
      ↓
Certificate
      ↓
Trusted Certificate Authority
      ↓
Identity Verification
```

---

## 3. Hashes

Password hashes are not intended to be used directly for normal authentication.

However, password hashes can be used for authentication in certain attacks.

Examples of these attacks are covered later in this room.

---

# 🎯 Authentication Goal

Regardless of the authentication material being used, the objective is:

```text
Prove to the domain
that you are
who you claim to be.
```

---

# 🔐 Authentication vs Authorisation

A common point of confusion is the difference between **authentication** and **authorisation**.

These are two separate processes.

---

## Authentication

Authentication:

> **Proves your identity.**

Example:

```text
"You are John."
```

Authentication answers:

> **Who are you?**

---

## Authorisation

Authorisation:

> **Determines what you are allowed to do.**

Example:

```text
"John has access to the finance share."
```

Authorisation answers:

> **What are you allowed to do?**

---

# 🔄 Authentication Comes First

Authentication always happens before authorisation.

The domain must first verify your identity before determining which resources you are permitted to access.

For example, when logging in to a domain-joined machine:

```text
User provides credentials
        ↓
Authentication
        ↓
Identity verified
        ↓
Authorisation
        ↓
Permissions evaluated
        ↓
Resource access
```

Once authenticated, authorisation checks can include:

- Group memberships
- Access Control Lists (ACLs)
- Resource permissions

These determine what the authenticated user can actually do on the network.

---

# 🌐 AD Authentication Protocols

When it comes to authentication in Active Directory, there are two core protocols that handle identity verification:

1. **NetNTLM / NTLM**
2. **Kerberos**

---

# 1️⃣ NetNTLM / NTLM

**NetNTLM**, commonly referred to as **NTLM**, is a:

> **Challenge-response authentication protocol**

It has been around since the early days of Windows NT.

The detailed NTLM authentication process is covered in the next task.

---

# 2️⃣ Kerberos

**Kerberos** is a:

> **Ticket-based authentication protocol**

It became the default authentication protocol in **Windows 2000** and remains the preferred authentication method today.

The detailed Kerberos authentication process is covered in the next task.

---

# 📜 Certificate-Based Authentication

Windows supports other authentication mechanisms, including:

> **Certificate-based TLS/SSL authentication for smart card logins**

These authentication mechanisms ultimately result in a **Kerberos ticket** being issued for further authentication to domain resources.

The certificate proves the user's identity, while Kerberos handles the actual session authentication afterwards.

### Authentication Flow

```text
Certificate
     ↓
Identity Proven
     ↓
Kerberos Ticket
     ↓
Authentication to Domain Resources
```

---

# 🌐 Other Protocols Encountered in AD

While working in Active Directory environments, you may also encounter protocols such as:

- **LDAP**
- **WebDAV**
- **SMB**

However, these are **service or directory access protocols**.

They rely on either:

```text
NTLM
```

or:

```text
Kerberos
```

to perform the actual authentication.

---

# 📂 Example: SMB

When accessing an SMB file share:

```text
Client
   ↓
SMB
   ↓
NTLM / Kerberos
   ↓
Authentication
   ↓
SMB File Share
```

The SMB protocol provides access to the file-sharing service, while NTLM or Kerberos handles the underlying authentication.

---

# 📁 Example: LDAP

When authenticating to an LDAP service:

```text
Client
   ↓
LDAP
   ↓
NTLM / Kerberos
   ↓
Authentication
   ↓
Directory Access
```

Again, LDAP provides directory access while NTLM or Kerberos can perform the underlying authentication.

---

# ⚠️ Why NTLM and Kerberos Matter

Understanding these two core authentication protocols is essential because nearly every Active Directory-based attack you encounter targets weaknesses in how:

```text
NTLM
   or
Kerberos
```

handles authentication.

The following tasks examine each protocol in detail and explain how they work.

---

# 📊 Quick Comparison

| Feature | NTLM | Kerberos |
|---|---|---|
| Authentication type | Challenge-response | Ticket-based |
| Introduced | Windows NT era | Default since Windows 2000 |
| Role in modern AD | Legacy / fallback | Preferred |
| Main focus of next tasks | NTLM authentication | Kerberos authentication |

---

# 💡 Key Takeaways

- Authentication proves who you are.
- Authorisation determines what you are allowed to do.
- Authentication happens before authorisation.
- Username and password are the most common authentication materials.
- Certificates can be used to prove identity.
- Password hashes can be used during certain authentication attacks.
- NTLM uses challenge-response authentication.
- Kerberos uses ticket-based authentication.
- Kerberos became the default authentication protocol in Windows 2000.
- LDAP, WebDAV, and SMB are service or directory access protocols that rely on NTLM or Kerberos for authentication.
- Understanding NTLM and Kerberos is fundamental to Active Directory security testing.

---

# 📚 References

- TryHackMe — Intro to AD Authentication