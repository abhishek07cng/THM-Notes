# 🔑 A07 - Authentication Failures

> **Category:** IAAA Failures
>
> Authentication Failures occur when an application cannot reliably verify or bind a user's identity, allowing attackers to impersonate legitimate users or gain unauthorized access.

---

# 📖 Overview

Authentication is the process of proving that a user is who they claim to be.

It answers one fundamental question:

> **"Can you prove your identity?"**

When authentication mechanisms are weak or implemented incorrectly, attackers may bypass login controls, hijack accounts, or impersonate other users.

---

# 🎯 Objectives

After studying this topic, you should understand:

- What authentication failures are
- Common authentication weaknesses
- How attackers exploit authentication logic
- The TryHackMe lab example
- Authentication testing methodology
- Prevention techniques

---

# 🧠 Common Authentication Failures

According to the TryHackMe room, common authentication failures include:

- Username Enumeration
- Weak or Guessable Passwords
- Missing Rate Limiting
- Registration/Login Logic Flaws
- Insecure Session Management
- Insecure Cookie Handling

---

# 🔍 Username Enumeration

Applications should never reveal whether a username exists.

Bad example:

```
Username does not exist
```

Good example:

```
Invalid username or password
```

Different responses allow attackers to discover valid usernames before attempting password attacks.

---

# 🔓 Weak Password Policies

Weak passwords significantly increase the risk of account compromise.

Examples:

```
123456

password

admin123

qwerty
```

Applications should enforce strong password requirements.

---

# 🚫 Missing Rate Limiting

If unlimited login attempts are allowed, attackers can automate password guessing.

Example attack:

```
admin

↓

password1

↓

password2

↓

password3

↓

...
```

Without account lockout or rate limiting, brute-force attacks become practical.

---

# ⚠️ Registration and Login Logic Flaws

The TryHackMe room demonstrates a logic flaw using usernames such as:

```
admin
```

and

```
aDmiN
```

If different parts of the application treat usernames differently (case-sensitive during registration but case-insensitive elsewhere), identity confusion can occur.

This can result in authentication being bound to the wrong account. :contentReference[oaicite:1]{index=1}

---

# 🍪 Session & Cookie Failures

Authentication does not end after login.

Applications must securely manage sessions.

Examples of insecure implementations include:

- Predictable session IDs
- Sessions not invalidated after logout
- Weak JWT validation
- Cookies without appropriate security attributes

---

# ⚙️ Authentication Flow

```
User

↓

Provides Username

↓

Provides Password

↓

Server Verifies Credentials

↓

Authentication Successful

↓

Session Created
```

Every stage should be securely implemented.

---

# 🧪 TryHackMe Lab Summary

In the TryHackMe exercise:

- The administrator username is known.
- The application allows registration using a differently cased username.
- This demonstrates a flaw where identity handling is inconsistent across the application.

The lab highlights the importance of consistent identity validation throughout registration and authentication. :contentReference[oaicite:2]{index=2}

---

# 🌍 Real-World Perspective

The uploaded material notes that authentication issues frequently arise from:

- Username enumeration
- Registration logic flaws
- Case normalization issues
- Session handling mistakes
- Cookie and JWT validation errors

It also highlights examples relevant to Node.js/MERN applications, including JWT algorithm validation and case-insensitive database lookups. :contentReference[oaicite:3]{index=3}

---

# 🐞 Bug Bounty Testing Methodology

When assessing authentication, test for:

- Username enumeration
- Password brute-force protections
- Account lockout mechanisms
- Registration validation
- Password reset logic
- Session fixation
- Session reuse
- Cookie security
- JWT handling
- MFA enforcement

---

# 🔍 Authentication Testing Checklist

- Can usernames be enumerated?
- Are passwords rate-limited?
- Can accounts be brute-forced?
- Are password reset tokens secure?
- Does logout invalidate sessions?
- Are cookies marked Secure and HttpOnly?
- Is MFA implemented correctly?
- Are JWTs validated correctly?

---

# ❌ Common Developer Mistakes

- Different error messages during login
- Weak password policies
- Missing account lockout
- Inconsistent username normalization
- Weak session management
- Trusting client-side authentication state
- Incorrect JWT validation

---

# 🛡️ Prevention

- Use generic authentication error messages.
- Enforce strong password policies.
- Implement rate limiting and account lockout.
- Normalize usernames consistently.
- Protect session cookies.
- Validate JWT signatures correctly.
- Require Multi-Factor Authentication for sensitive accounts.
- Invalidate sessions on logout.

---

# 📝 Key Takeaways

- Authentication proves identity.
- Authentication should never rely solely on passwords.
- Registration and login must follow identical validation rules.
- Secure session management is as important as secure login.
- Consistent identity handling prevents account confusion.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Authentication Cheat Sheet