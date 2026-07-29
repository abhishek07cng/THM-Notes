# 🔐 IAAA (Identity, Authentication, Authorization & Accountability)

> Before understanding the OWASP Top 10 (2025), it is essential to understand **IAAA**. Every secure application relies on these four security principles to ensure that users are properly identified, verified, authorized, and monitored.

---

# 📖 What is IAAA?

**IAAA** is a security model that describes how applications manage users and their actions.

The four components are:

```
Identity
      ↓
Authentication
      ↓
Authorization
      ↓
Accountability
```

Each step depends on the previous one.

If one layer fails, the security of the entire application is weakened.

---

# 🔄 Authentication Flow

```
                User
                  │
                  ▼
        Identity (Who are you?)
                  │
                  ▼
Authentication (Prove who you are)
                  │
                  ▼
 Authorization (What can you do?)
                  │
                  ▼
Accountability (Record every action)
```

---

# 1️⃣ Identity

## Definition

Identity represents **who the user claims to be**.

It is simply a unique identifier that distinguishes one user from another.

Examples include:

- Username
- Email Address
- Employee ID
- Student ID
- Customer ID
- UUID
- Service Account

Identity **does not prove ownership**.

It only answers:

> **"Who are you?"**

---

## Examples

```
Username : abhishek

Email : john@example.com

Employee ID : EMP1023

UUID : 34da98e8-ef3b...
```

These values identify a user but do not verify them.

---

# 2️⃣ Authentication

## Definition

Authentication is the process of verifying that a user really owns the claimed identity.

It answers:

> **"Can you prove that you are this user?"**

---

## Common Authentication Methods

### Password

```
Username
Password
```

---

### OTP (One-Time Password)

```
Username

↓

Password

↓

OTP
```

---

### Multi-Factor Authentication (MFA)

Something you know

+

Something you have

+

Something you are

Example:

Password

+

Authenticator App

+

Fingerprint

---

### Passkeys

Modern passwordless authentication using cryptographic keys.

Examples:

- Face ID
- Fingerprint
- Windows Hello

---

## Authentication Failure Example

```
Login

↓

Username : admin

Password : admin123
```

If the application allows weak passwords or unlimited login attempts, attackers may successfully brute-force the account.

---

# 3️⃣ Authorization

## Definition

Authorization determines **what an authenticated user is allowed to access or perform**.

It answers:

> **"What can you do?"**

---

## Example

Imagine three users.

```
Admin

↓

Can create users
Can delete users
Can manage system

------------------------

Moderator

↓

Can edit posts
Can remove comments

------------------------

User

↓

Can only edit own profile
```

Even though all users are authenticated, each has different permissions.

---

## Authorization Example

```
User A

↓

/profile?id=101

↓

Own profile ✔
```

Changing the request to

```
/profile?id=102
```

should **NOT** reveal another user's data.

If it does,

➡️ Broken Access Control

➡️ IDOR

---

# 4️⃣ Accountability

## Definition

Accountability ensures that every important action is recorded.

It answers:

> **"Who performed what action, when, and from where?"**

---

Typical logged information includes:

- Username
- Timestamp
- IP Address
- Device
- Browser
- Action performed
- Success / Failure

---

Example

```
[12:30 PM]

User: admin

IP: 192.168.1.10

Action:
Deleted User #245
```

This enables auditing and incident investigation.

---

# 🔗 Relationship Between the Four Components

```
Identity

↓

Authentication

↓

Authorization

↓

Accountability
```

You **cannot skip** a stage.

For example:

❌ Without Identity

↓

Authentication is impossible.

---

Without Authentication

↓

Authorization cannot be trusted.

---

Without Authorization

↓

Users may access resources they should never see.

---

Without Accountability

↓

Security incidents become difficult to investigate.

---

# 🌍 Real-World Example

Imagine logging into your online banking application.

## Identity

```
Customer ID
```

↓

## Authentication

```
Password

+

OTP
```

↓

## Authorization

```
Can only access YOUR account.
```

↓

## Accountability

```
Transfer of ₹50,000

↓

Logged with

Time

IP

Location

Device
```

---

# 🔥 Why IAAA Matters

Most critical web vulnerabilities happen because one or more of these four areas are implemented incorrectly.

Examples include:

| Failure | Result |
|----------|--------|
| Weak Authentication | Account Takeover |
| Missing Authorization | IDOR / Privilege Escalation |
| Poor Accountability | Attacks remain undetected |
| Identity Confusion | User impersonation |

---

# 🐞 Bug Bounty Perspective

When testing an application, always ask:

### Identity

- Can usernames be duplicated?
- Can email addresses collide?
- Is case sensitivity handled correctly?

---

### Authentication

- Can passwords be brute-forced?
- Is MFA enforced?
- Are sessions secure?
- Can authentication be bypassed?

---

### Authorization

- Can I access another user's data?
- Can I modify IDs?
- Can I access admin endpoints?
- Can I bypass role checks?

---

### Accountability

- Are login failures logged?
- Are admin actions recorded?
- Can attackers delete logs?
- Are suspicious activities alerted?

---

# 📝 Key Takeaways

- Identity tells the application **who you are**.
- Authentication proves your identity.
- Authorization determines your permissions.
- Accountability records your actions.

These four principles form the security foundation of modern web applications.

Understanding IAAA makes it easier to recognize vulnerabilities such as:

- Broken Access Control
- Authentication Failures
- Logging & Alerting Failures
- Privilege Escalation
- IDOR
- Account Takeover

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Foundation