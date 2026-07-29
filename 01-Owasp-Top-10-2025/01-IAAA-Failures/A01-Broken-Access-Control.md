# 🔓 A01 - Broken Access Control

> **Category:** IAAA Failures
>
> Broken Access Control occurs when an application fails to properly enforce authorization rules, allowing users to perform actions or access resources beyond their intended permissions.

---

# 📖 Overview

Access Control determines **what an authenticated user is allowed to do**.

Even if a user has successfully logged in, they should only be able to access the resources and functions they are explicitly authorized to use.

Broken Access Control occurs when the application trusts user-supplied requests without verifying whether the user has permission to perform the requested action.

This is one of the most common and impactful web application vulnerabilities because it can expose sensitive information or lead to complete account compromise.

---

# 🎯 Objectives

After completing this note, you should understand:

- What Broken Access Control is
- How authorization differs from authentication
- Horizontal vs Vertical Privilege Escalation
- IDOR, BOLA and BFLA
- How to identify Broken Access Control during testing
- Common developer mistakes
- Secure implementation strategies

---

# 🧠 Understanding Authorization

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to do?**

Example:

```
Login

↓

User: Alice

↓

Authenticated ✅

↓

Authorization Check

↓

Allowed:
View own profile

Not Allowed:
View Bob's profile
```

Authentication alone is **not enough**.

Every request must be authorized.

---

# 🚨 What is Broken Access Control?

Broken Access Control occurs when the application does not verify permissions on every request.

Instead, it assumes:

> "The user is already logged in, so this request must be valid."

Attackers exploit this assumption by modifying requests.

---

# 🔥 Example

Original request

```
GET /account?id=25
```

Attacker changes it to

```
GET /account?id=26
```

If Account #26 belongs to another user and the application returns its data,

➡️ Broken Access Control

---

# 📂 Types of Broken Access Control

## 1️⃣ Horizontal Privilege Escalation

A user accesses another user with the same privileges.

Example

```
User A

↓

/profile?id=100
```

↓

```
/profile?id=101
```

Accessing another customer's profile.

---

## 2️⃣ Vertical Privilege Escalation

A normal user gains administrator functionality.

Example

```
User

↓

/admin
```

↓

Admin Dashboard

This should never be accessible without proper authorization.

---

## 3️⃣ Insecure Direct Object Reference (IDOR)

One of the most common Broken Access Control vulnerabilities.

The application directly exposes identifiers:

```
?id=1

?id=2

?id=3
```

If changing the identifier exposes another user's object,

IDOR exists.

---

## 4️⃣ BOLA (Broken Object Level Authorization)

API equivalent of IDOR.

Example

```
GET

/api/orders/100
```

↓

```
GET

/api/orders/101
```

If another user's order is returned,

Broken Object Level Authorization exists.

---

## 5️⃣ BFLA (Broken Function Level Authorization)

The application exposes administrative functionality without verifying the user's role.

Example

```
POST

/api/admin/delete-user
```

If a normal user can access this endpoint,

Broken Function Level Authorization exists.

---

# ⚙️ How the Vulnerability Works

```
User

↓

Logs In

↓

Authenticated

↓

Changes Request

↓

Server trusts request

↓

No authorization check

↓

Sensitive Data Returned
```

---

# 🧪 TryHackMe Lab Summary

The TryHackMe room demonstrates an account lookup where changing the `accountID` value in the URL allows access to another user's account information.

The exercise highlights how authentication alone is insufficient; the server must verify ownership of every requested object before returning data. :contentReference[oaicite:0]{index=0}

---

# 🌍 Real-World Perspective

According to the uploaded TryHackMe material:

- IDOR is one of the highest-volume bug bounty findings.
- Testing commonly involves two accounts.
- APIs are especially vulnerable because object identifiers are frequently exposed.
- OWASP API Security refers to this issue as **Broken Object Level Authorization (BOLA)**. :contentReference[oaicite:1]{index=1}

---

# 🐞 Bug Bounty Testing Methodology

When testing an application, check whether you can:

- Change numeric IDs
- Change UUIDs
- Access another user's profile
- View another invoice
- Download another document
- Modify another user's order
- Access admin-only APIs
- Change role parameters
- Bypass hidden frontend controls

Always compare responses between **two separate test accounts**.

---

# 🔍 Common Endpoints to Test

```
/profile?id=
/user/
/account/
/invoice/
/orders/
/documents/
/download/
/api/users/
/api/orders/
/admin/
```

---

# 💻 Sample Requests

Original request:

```http
GET /profile?id=100
```

Modified request:

```http
GET /profile?id=101
```

JSON example:

```json
{
  "userId": 25
}
```

Modified:

```json
{
  "userId": 26
}
```

---

# ❌ Common Developer Mistakes

- Trusting user-controlled IDs
- Performing authentication without authorization
- Hiding admin functionality only in the frontend
- Missing ownership checks
- Assuming API endpoints are inaccessible
- Relying solely on client-side validation

---

# 🛡️ Prevention

- Validate authorization on **every request**
- Verify object ownership before returning data
- Apply Role-Based Access Control (RBAC)
- Use the Principle of Least Privilege
- Perform server-side authorization checks
- Never trust client-supplied identifiers

---

# 📝 Key Takeaways

- Authentication does **not** equal authorization.
- Every request must be checked for permission.
- IDOR is a common example of Broken Access Control.
- API environments often refer to this as BOLA or BFLA.
- Broken Access Control remains one of the most frequently discovered bug bounty vulnerabilities.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Top 10
- OWASP API Security Top 10