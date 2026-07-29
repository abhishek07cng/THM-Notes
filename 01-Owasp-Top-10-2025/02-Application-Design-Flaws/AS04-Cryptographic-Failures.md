# 🔐 AS04 - Cryptographic Failures

> **Category:** Application Design Flaws
>
> Cryptographic Failures occur when sensitive information is not properly protected through encryption, hashing, key management, or secure communication.

---

# 📖 Overview

Modern applications process sensitive information every day:

- User passwords
- Credit card numbers
- Personal information
- API keys
- Authentication tokens
- Session cookies
- Encryption keys

If this information is stored or transmitted insecurely, attackers may gain unauthorized access.

---

# 🎯 Learning Objectives

After completing this topic, you should understand:

- What Cryptographic Failures are
- Difference between hashing and encryption
- Weak cryptographic algorithms
- Password hashing best practices
- Secret management
- Secure communications
- Prevention techniques

---

# 🧠 What are Cryptographic Failures?

Cryptographic Failures happen when sensitive data is not adequately protected because encryption or hashing is missing, implemented incorrectly, or relies on weak algorithms.

Examples include:

- Passwords stored in plaintext
- Weak hashing algorithms
- Weak encryption
- Hardcoded secrets
- Exposed encryption keys
- Insecure TLS configuration

---

# 🔍 Common Cryptographic Failures

## 1️⃣ Plaintext Password Storage

❌ Bad

```
Database

username

alice

password

Password123
```

If the database is leaked, every password is immediately exposed.

---

## 2️⃣ Weak Password Hashing

Weak algorithms include:

```
MD5

SHA-1
```

These algorithms are considered unsuitable for password storage because they are fast and vulnerable to brute-force attacks.

---

## 3️⃣ Strong Password Hashing

The TryHackMe room recommends using slow password hashing algorithms such as:

- bcrypt
- scrypt
- Argon2

These algorithms are specifically designed to make password cracking significantly more difficult. :contentReference[oaicite:1]{index=1}

---

## 4️⃣ Weak Encryption Algorithms

Examples of outdated algorithms include:

```
DES
```

Applications should instead use modern, well-reviewed cryptographic libraries and algorithms.

---

## 5️⃣ Hardcoded Secrets

Never store:

```
API Keys

Database Passwords

Cloud Credentials

JWT Secrets
```

inside:

```
Source Code

Git Repository

Docker Image
```

Attackers frequently search public repositories for exposed secrets.

---

## 6️⃣ Rolling Your Own Cryptography

One of the key points from the TryHackMe room is:

> Never create your own encryption algorithm.

Instead, rely on well-established, industry-tested cryptographic libraries and implementations. :contentReference[oaicite:2]{index=2}

---

# 🔑 Hashing vs Encryption

## Hashing

```
Password

↓

Hash Function

↓

Hash Stored
```

Properties:

- One-way operation
- Cannot be reversed directly
- Used for passwords

---

## Encryption

```
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

Properties:

- Reversible
- Used to protect confidential information

---

# 🌐 Protecting Data in Transit

Sensitive information should also be protected while travelling across networks.

Examples:

- HTTPS
- TLS

Without secure transport, attackers may intercept sensitive communications.

---

# 🧪 TryHackMe Summary

The TryHackMe room highlights several common cryptographic mistakes:

- Passwords stored without hashing
- Weak algorithms such as MD5, SHA-1, and DES
- Exposed credentials and secrets
- Custom cryptographic implementations
- Failure to use trusted cryptographic libraries

It recommends using modern password hashing algorithms and secure secret management. :contentReference[oaicite:3]{index=3}

---

# 🌍 Real-World Examples

Examples frequently encountered during security reviews include:

- Database backups containing plaintext passwords
- API keys committed to Git repositories
- Hardcoded cloud credentials
- JWT signing secrets exposed in configuration files
- Sensitive traffic transmitted over HTTP instead of HTTPS

---

# 🐞 Bug Bounty Perspective

Common findings include:

- Secrets exposed in JavaScript files
- API keys committed to public repositories
- Weak password reset implementations
- Sensitive information transmitted over insecure channels
- Hardcoded credentials in mobile applications

Always validate impact before reporting and follow the programme's disclosure policy.

---

# 🛠️ Secure Secret Management

Avoid storing secrets directly in source code.

Instead use:

- Environment variables
- Secret management services
- Dedicated credential vaults

Rotate secrets regularly and restrict access using the principle of least privilege.

---

# ❌ Common Developer Mistakes

- Storing passwords in plaintext.
- Using MD5 or SHA-1 for password storage.
- Creating custom encryption algorithms.
- Embedding secrets in source code.
- Reusing encryption keys.
- Ignoring TLS configuration.
- Exposing credentials through version control.

---

# 🛡️ Prevention

- Hash passwords using bcrypt, scrypt, or Argon2.
- Never store plaintext passwords.
- Use modern, trusted cryptographic libraries.
- Protect secrets with secure secret management.
- Use HTTPS/TLS for sensitive communications.
- Rotate keys regularly.
- Restrict access to encryption keys.
- Review cryptographic implementations during security assessments.

---

# ✅ Cryptography Checklist

- [ ] Passwords hashed securely.
- [ ] No plaintext credentials.
- [ ] Modern hashing algorithms used.
- [ ] Strong encryption algorithms used.
- [ ] Secrets stored outside source code.
- [ ] HTTPS enforced.
- [ ] Keys rotated periodically.
- [ ] Custom cryptography avoided.

---

# 💡 Key Takeaways

- Cryptography protects sensitive information.
- Password hashing and encryption solve different problems.
- MD5, SHA-1, and DES should not be used for new applications.
- Never implement your own cryptographic algorithms.
- Secure key and secret management is as important as strong encryption.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Cryptographic Storage Guidance