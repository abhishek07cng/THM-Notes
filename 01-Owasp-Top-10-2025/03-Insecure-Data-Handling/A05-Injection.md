# 💉 A05 – Injection

> **Category:** Insecure Data Handling
>
> Injection vulnerabilities occur when an application accepts untrusted input and passes it directly to an interpreter such as a database, operating system shell, template engine, or API without proper validation or separation.

---

# 📖 Overview

Injection has remained one of the most dangerous vulnerabilities in web application security for many years.

Instead of treating user input as **data**, vulnerable applications accidentally treat it as **code or commands**, allowing attackers to change how the application behaves.

Because of its high impact and widespread occurrence, Injection continues to appear on the OWASP Top 10 list. :contentReference[oaicite:1]{index=1}

---

# 🎯 Learning Objectives

After studying this topic, you should understand:

- What Injection vulnerabilities are
- Why Injection occurs
- Common Injection types
- How attackers exploit Injection
- Secure coding practices
- Prevention techniques

---

# 🧠 What is Injection?

Injection occurs when an application takes user input and mishandles it.

Instead of processing the input safely, the application passes it directly into a system capable of executing commands or queries.

Examples include:

- Database engines
- Operating system shells
- Template engines
- APIs

This allows attacker-controlled input to become executable instructions. :contentReference[oaicite:2]{index=2}

---

# ⚙️ Basic Attack Flow

```text
User Input

↓

Application

↓

No Validation

↓

Interpreter

↓

Attacker-Controlled Command Executes
```

---

# 🔍 Common Types of Injection

The TryHackMe room highlights several common Injection vulnerabilities.

---

## 1️⃣ SQL Injection (SQLi)

SQL Injection occurs when user input becomes part of a database query.

Example:

```sql
SELECT * FROM users
WHERE username = '<user_input>';
```

If the application concatenates input directly into the query, attackers may modify the SQL statement.

SQL Injection can lead to:

- Authentication bypass
- Data disclosure
- Data modification
- Database compromise

---

## 2️⃣ Command Injection

Applications sometimes execute operating system commands.

Unsafe example:

```text
ping <user_input>
```

If user input is passed directly to the shell, attackers may execute additional operating system commands.

Possible impact:

- Remote Code Execution (RCE)
- File access
- Reverse shells
- Complete server compromise

---

## 3️⃣ Server-Side Template Injection (SSTI)

Template engines render dynamic web pages.

The TryHackMe room provides the following example:

```text
{{ 7 * 7 }}
```

which evaluates to:

```text
49
```

If attacker input is interpreted by the template engine, arbitrary template expressions may execute. :contentReference[oaicite:3]{index=3}

---

## 4️⃣ AI Prompt Injection

The room also highlights **AI Prompt Injection** as a modern form of Injection.

Instead of targeting SQL or operating system commands, attackers manipulate an AI application's instructions by supplying crafted prompts.

Potential consequences include:

- Ignoring intended instructions
- Revealing confidential information
- Producing unauthorised outputs

This reflects how modern AI-integrated applications can be vulnerable when untrusted input is treated as trusted instructions. :contentReference[oaicite:4]{index=4}

---

# ⚠️ Why Injection Happens

Common causes include:

- String concatenation
- Missing input validation
- Missing input sanitisation
- Passing user input directly to interpreters
- Trusting user-controlled data

---

# 🌍 Real-World Examples

Examples commonly encountered include:

- Login forms vulnerable to SQL Injection
- File upload features vulnerable to Command Injection
- Search functionality vulnerable to SQL Injection
- Server-rendered templates vulnerable to SSTI
- AI assistants vulnerable to Prompt Injection

---

# 🐞 Bug Bounty Perspective

Injection vulnerabilities remain among the highest-impact findings.

Common targets include:

- Login pages
- Search functionality
- Admin panels
- File conversion services
- Reporting systems
- AI-powered chat interfaces
- API endpoints

Injection findings often result in:

- Account takeover
- Sensitive data exposure
- Remote Code Execution
- Full application compromise

---

# 🧪 TryHackMe Summary

The TryHackMe room explains that Injection occurs when applications fail to safely handle user input before passing it to another system.

Examples covered include:

- SQL Injection
- Command Injection
- AI Prompt Injection
- Server-Side Template Injection (SSTI)

It emphasises that these remain highly severe vulnerabilities and should be treated accordingly. :contentReference[oaicite:5]{index=5}

---

# 🛡️ How to Prevent Injection

The TryHackMe room recommends treating **all user input as untrusted**.

For SQL:

- Use prepared statements.
- Use parameterised queries.
- Avoid building SQL queries using string concatenation.

For operating system commands:

- Avoid functions that invoke the system shell directly.
- Prefer safe APIs that do not execute shell commands.

Input validation and sanitisation should also be applied before the application processes user input. This includes escaping dangerous characters, enforcing expected data types, and filtering unexpected input. :contentReference[oaicite:6]{index=6}

---

# 🔒 Secure Development Checklist

- [ ] Prepared SQL statements used.
- [ ] Parameterised queries implemented.
- [ ] No shell execution with user input.
- [ ] Input validation implemented.
- [ ] Input sanitisation performed.
- [ ] Dangerous characters escaped.
- [ ] Strict data types enforced.
- [ ] User input always treated as untrusted.

---

# ❌ Common Developer Mistakes

- Building SQL queries with string concatenation.
- Passing user input directly to shell commands.
- Trusting user input.
- Missing input validation.
- Missing sanitisation.
- Treating AI prompts as trusted instructions.
- Rendering user-controlled template expressions.

---

# 💡 Key Takeaways

- Injection occurs when untrusted input becomes executable instructions.
- SQL Injection, Command Injection, SSTI, and AI Prompt Injection are all forms of Injection.
- Prepared statements and parameterised queries are the preferred defence against SQL Injection.
- Avoid invoking operating system shells with user-controlled input.
- Always validate and sanitise user input before it reaches an interpreter.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)