# 🔐 A04 – Cryptographic Failures

> **Category:** Insecure Data Handling
>
> Cryptographic Failures occur when sensitive information is not properly protected because encryption is missing, implemented incorrectly, or relies on weak cryptographic practices.

---

# 📖 Overview

Modern web applications constantly handle sensitive information such as:

- Passwords
- Session Tokens
- API Keys
- Personal Information (PII)
- Payment Details
- JWT Secrets
- Database Credentials
- Encryption Keys

If these are not protected correctly, attackers may steal confidential information or completely compromise an application.

---

# 🎯 Learning Objectives

After studying this topic you should understand:

- What Cryptographic Failures are
- Why cryptography is important
- Common implementation mistakes
- Weak cryptographic algorithms
- Secure password hashing
- Secret management
- Prevention techniques

---

# 🧠 What are Cryptographic Failures?

Cryptographic failures happen when sensitive data isn't adequately protected due to:

- Lack of encryption
- Faulty implementation
- Weak cryptographic algorithms
- Poor key management
- Insecure storage of secrets
- Failure to secure data during transmission

Examples include:

- Passwords stored without hashing
- Using MD5 or SHA-1
- Hardcoded encryption keys
- Data transmitted without HTTPS

---

# ⚠️ Why It Matters

Web applications rely on cryptography everywhere:

- Protecting network traffic
- Securing stored data
- Verifying identities
- Protecting API tokens
- Safeguarding credentials
- Encrypting sensitive files

When cryptographic protections fail, attackers may gain access to passwords, tokens, personal information, or confidential business data. :contentReference[oaicite:1]{index=1}

---

# 🔍 Common Cryptographic Failures

## 1️⃣ Plaintext Password Storage

❌ Bad

```text
username

alice

password

Password123
```

If the database is compromised, every user's password is immediately exposed.

---

## 2️⃣ Weak Hashing Algorithms

Examples:

```text
MD5

SHA-1
```

These algorithms are considered unsuitable for password storage because they are extremely fast and vulnerable to brute-force attacks. :contentReference[oaicite:2]{index=2}

---

## 3️⃣ Weak Encryption

Outdated algorithms include:

```text
DES
```

Applications should instead use modern, industry-approved cryptographic algorithms and libraries.

---

## 4️⃣ Hardcoded Secrets

Never embed:

- API Keys
- JWT Secrets
- Database Passwords
- Cloud Credentials

inside:

```text
Source Code

Configuration Files

Git Repositories
```

If source code becomes public or is leaked, every embedded secret becomes exposed. The TryHackMe room specifically advises against embedding third-party service credentials in source code or repositories. :contentReference[oaicite:3]{index=3}

---

## 5️⃣ Poor Key Management

Examples include:

- Encryption keys stored beside encrypted data
- Keys never rotated
- Shared keys used everywhere
- Unrestricted access to secrets

Weak key management often defeats otherwise strong encryption.

---

## 6️⃣ Unencrypted Data in Transit

Sensitive information should never travel over plain HTTP.

Always protect communication using:

- HTTPS
- TLS

Otherwise attackers performing network interception may capture credentials or sensitive information.

---

## 7️⃣ "Rolling Your Own Cryptography"

One of the strongest recommendations from the TryHackMe room is:

> Never create your own encryption algorithm.

Instead, rely on well-established, vetted, industry-standard cryptographic libraries and algorithms. :contentReference[oaicite:4]{index=4}

---

# 🔑 Hashing vs Encryption

## Hashing

```text
Password

↓

Hash Function

↓

Hash Stored
```

Characteristics:

- One-way operation
- Used for password storage
- Cannot be reversed directly

---

## Encryption

```text
Sensitive Data

↓

Encryption Key

↓

Ciphertext

↓

Decryption Key

↓

Original Data
```

Characteristics:

- Reversible
- Used to protect confidential information

---

# 🔒 Secure Password Hashing

The TryHackMe room recommends using slow password hashing algorithms specifically designed for credential storage:

- bcrypt
- scrypt
- Argon2

These significantly increase the cost of brute-force attacks compared with general-purpose hash functions. :contentReference[oaicite:5]{index=5}

---

# 🛠️ Secure Secret Management

Instead of storing credentials in code, use:

- Environment Variables
- Secret Management Systems
- Dedicated Secret Vaults

Examples include cloud key management and secret storage platforms.

---

# ⚙️ Attack Flow

```text
Application

↓

Sensitive Data

↓

Weak / Missing Cryptography

↓

Attacker Obtains Data

↓

Account Takeover / Data Breach
```

---

# 🧪 TryHackMe Summary

The TryHackMe room highlights several common cryptographic mistakes:

- Passwords stored without hashing
- Weak algorithms such as MD5, SHA-1, and DES
- Exposed encryption keys
- Hardcoded credentials
- Creating custom cryptographic algorithms

It recommends modern password hashing algorithms, trusted cryptographic libraries, and secure secret management. :contentReference[oaicite:6]{index=6}

---

# 🌍 Real-World Examples

Examples frequently encountered during security assessments include:

- Public Git repositories containing API keys
- Hardcoded JWT signing secrets
- Database backups with plaintext passwords
- Internal services running without HTTPS
- Configuration files containing cloud credentials

---

# 🐞 Bug Bounty Perspective

Cryptographic weaknesses often appear as:

- Secrets exposed in JavaScript files
- API keys committed to Git repositories
- Weak password storage
- Sensitive information transmitted over HTTP
- JWT secrets leaked through configuration files

These issues frequently lead to account compromise or unauthorised access.

---

# ❌ Common Developer Mistakes

- Using MD5 or SHA-1 for passwords.
- Storing plaintext passwords.
- Hardcoding secrets.
- Creating custom encryption algorithms.
- Never rotating encryption keys.
- Ignoring HTTPS.
- Committing secrets to version control.

---

# 🛡️ Prevention

- Use bcrypt, scrypt, or Argon2 for password hashing.
- Use trusted cryptographic libraries.
- Never create custom cryptographic algorithms.
- Protect all sensitive communications with HTTPS/TLS.
- Store secrets in dedicated secret management systems.
- Rotate keys regularly.
- Audit cryptographic implementations periodically.

---

# ✅ Security Checklist

- [ ] Passwords hashed securely.
- [ ] No plaintext credentials.
- [ ] Modern cryptographic algorithms used.
- [ ] Secrets stored outside source code.
- [ ] HTTPS enforced.
- [ ] Encryption keys protected.
- [ ] Keys rotated regularly.
- [ ] Custom cryptography avoided.

---

# 💡 Key Takeaways

- Cryptographic failures usually result from misuse rather than broken algorithms.
- MD5, SHA-1, and DES should not be used for modern applications.
- Passwords should be hashed using bcrypt, scrypt, or Argon2.
- Secrets should never be embedded in source code.
- Proper key management is just as important as strong encryption.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)