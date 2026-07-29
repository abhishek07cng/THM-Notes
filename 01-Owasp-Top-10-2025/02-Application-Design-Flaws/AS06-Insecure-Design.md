# 🏛️ AS06 - Insecure Design

> **Category:** Application Design Flaws
>
> Insecure Design refers to security weaknesses that originate from the application's architecture, workflow, or business logic rather than coding mistakes.

---

# 📖 Overview

An application can be written using secure programming practices and still be vulnerable.

Why?

Because security begins **before code is written**.

Poor architectural decisions can introduce vulnerabilities that cannot simply be fixed with input validation or patches.

Examples include:

- Business logic flaws
- Missing authorization rules
- Race conditions
- Client-side trust
- Poor workflow design
- Missing threat modelling

---

# 🎯 Learning Objectives

After studying this topic you should understand:

- What Insecure Design means
- Why secure coding alone is insufficient
- Business logic vulnerabilities
- Race conditions
- Client-side trust issues
- Threat modelling
- AI-related design risks

---

# 🧠 What is Insecure Design?

Insecure Design occurs when the application's intended behaviour allows attackers to achieve actions that should never have been possible.

Unlike implementation vulnerabilities:

```
Implementation Bug

↓

Developer writes insecure code
```

Insecure Design:

```
Architecture

↓

Application designed insecurely

↓

Every implementation follows that design
```

---

# 🔍 Secure Design vs Secure Coding

Secure Coding asks:

> "Is this code written securely?"

Secure Design asks:

> "Should this feature work this way in the first place?"

Both are necessary.

---

# ⚠️ Common Insecure Design Examples

## 1️⃣ Business Logic Vulnerabilities

The application behaves exactly as designed.

The problem is that the design itself is insecure.

Example:

```
Coupon

↓

Redeem

↓

Redeem Again

↓

Redeem Again

↓

Unlimited Discounts
```

The application never prevents multiple redemptions.

---

## 2️⃣ Race Conditions

Two requests execute simultaneously.

Example:

```
Request A

↓

Request B

↓

Both processed

↓

Balance becomes negative
```

The logic assumes only one request is processed at a time.

---

## 3️⃣ Client-Side Trust

Never trust calculations performed in the browser.

Example:

```
Price:

₹1000
```

Attacker changes:

```
₹100
```

If the server accepts the modified value,

the application has an insecure design.

---

## 4️⃣ Password Reset Design

A password reset workflow should never expose reset tokens.

The TryHackMe material highlights an example where a password reset token is both emailed **and returned in the JSON response "for debugging"**, making account takeover possible despite technically working as designed. :contentReference[oaicite:1]{index=1}

---

## 5️⃣ Missing Threat Modelling

Applications should identify threats before implementation.

Questions include:

- Who are the attackers?
- What assets require protection?
- Which actions should never be possible?
- What assumptions are unsafe?

Without threat modelling, insecure workflows often remain unnoticed.

---

# 🤖 AI Security Perspective

The uploaded TryHackMe room includes modern examples involving AI-enabled applications.

Examples include:

- Prompt injection against customer-support chatbots.
- AI systems leaking system prompts.
- AI agents revealing other users' information.
- AI performing actions beyond intended permissions.

These are examples of insecure system design rather than traditional implementation bugs. :contentReference[oaicite:2]{index=2}

---

# 📦 Autonomous Workflow Risks

The room also references the LiteLLM incident as an example where weaknesses in automated development workflows contributed to a software supply chain compromise.

The lesson is that automation should include:

- Human approval gates
- Secure credential management
- Deployment validation
- Least privilege

before production releases. :contentReference[oaicite:3]{index=3}

---

# ⚙️ Attack Flow

```
Application

↓

Insecure Business Logic

↓

Attacker Understands Workflow

↓

Abuses Intended Behaviour

↓

Unauthorised Result
```

---

# 🧪 TryHackMe Summary

The TryHackMe room demonstrates that many serious vulnerabilities originate from poor design decisions rather than programming mistakes.

Examples include:

- Coupon redemption abuse
- Password reset workflow flaws
- Client-side price calculation
- AI prompt injection
- Autonomous deployment risks

The room emphasises that these vulnerabilities usually require human reasoning to identify rather than automated scanning. :contentReference[oaicite:4]{index=4}

---

# 🌍 Real-World Examples

Examples frequently observed during security assessments include:

- Checkout prices trusted from the browser.
- Unlimited reward point redemption.
- Reusable discount coupons.
- Bank transfers allowing negative balances.
- Password reset tokens exposed through APIs.
- AI assistants performing privileged operations without sufficient validation.

---

# 🐞 Bug Bounty Perspective

Many high-severity bug bounty reports involve insecure design.

Common examples include:

- Business logic flaws.
- Race conditions.
- Price manipulation.
- Loyalty point abuse.
- Workflow bypasses.
- AI prompt injection.
- Missing approval processes.

These issues are difficult for automated scanners to detect because they depend on understanding how the application is intended to behave.

---

# 🔍 Testing Methodology

When reviewing an application:

- Understand the business workflow.
- Ask what assumptions the application makes.
- Test whether limits can be bypassed.
- Attempt actions in unexpected sequences.
- Use multiple accounts where appropriate.
- Consider concurrent requests for race conditions.
- Verify that all sensitive decisions are enforced on the server.

---

# ❌ Common Developer Mistakes

- Trusting client-side calculations.
- Assuming users follow the intended workflow.
- Not modelling abuse cases.
- Missing rate limits.
- Ignoring concurrent execution.
- Allowing automation without approval.
- Designing features without security review.

---

# 🛡️ Prevention

- Perform threat modelling early.
- Design with abuse cases in mind.
- Validate critical decisions on the server.
- Apply rate limiting where appropriate.
- Protect workflows against race conditions.
- Review business logic during testing.
- Include security reviews during system design.
- Require human approval for sensitive automated actions.

---

# ✅ Secure Design Checklist

- [ ] Threat modelling completed.
- [ ] Abuse cases documented.
- [ ] Business rules validated server-side.
- [ ] Client input never trusted.
- [ ] Race conditions considered.
- [ ] Security review included during design.
- [ ] High-risk automation requires approval.
- [ ] AI workflows reviewed for prompt injection and data leakage.

---

# 💡 Key Takeaways

- Insecure Design is an architectural problem, not just a coding problem.
- Secure coding cannot compensate for insecure workflows.
- Business logic flaws often have significant real-world impact.
- Threat modelling should occur before implementation.
- Human review remains essential for identifying complex design weaknesses.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Secure by Design Principles