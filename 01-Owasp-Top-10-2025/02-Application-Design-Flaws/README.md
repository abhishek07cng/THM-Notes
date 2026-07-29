# 🏗️ Application Design Flaws

The **Application Design Flaws** category in the TryHackMe **OWASP Top 10 (2025)** room focuses on security weaknesses introduced during the design, architecture, deployment, and maintenance of an application.

Unlike implementation bugs, these issues often originate from insecure design decisions, unsafe defaults, poor dependency management, or weak operational practices.

These flaws can affect an entire application even when the source code itself is correct.

---

# 🎯 Learning Objectives

By completing this section, you will learn to:

- Understand common application design weaknesses.
- Identify insecure configurations.
- Recognize supply chain risks.
- Understand secure cryptographic design.
- Identify insecure design patterns.
- Apply secure-by-design principles during development.
- Recognize these weaknesses during penetration testing and bug bounty hunting.

---

# 📚 Topics Covered

## AS02 – Security Misconfiguration

Improper or insecure application, server, cloud, container, or framework configuration.

Topics include:

- Default credentials
- Debug mode
- Directory listing
- Cloud misconfiguration
- Docker mistakes
- Kubernetes exposure
- Information disclosure

---

## AS03 – Software Supply Chain Failures

Security risks introduced through third-party software and dependencies.

Topics include:

- Vulnerable libraries
- Malicious packages
- Dependency confusion
- Typosquatting
- CI/CD compromise
- Package poisoning

---

## AS04 – Cryptographic Failures

Improper protection of sensitive data.

Topics include:

- Weak hashing
- Weak encryption
- Poor key management
- Password storage
- Secure communication

---

## AS06 – Insecure Design

Security problems introduced during application architecture.

Topics include:

- Missing threat modeling
- Business logic flaws
- Race conditions
- Client-side trust
- Poor authorization design
- AI-assisted application risks

---

# 🔄 Relationship Between These Topics

```
Poor Design
      │
      ▼
Unsafe Configuration
      │
      ▼
Weak Cryptography
      │
      ▼
Supply Chain Risks
      │
      ▼
Application Compromise
```

---

# 🛡️ Secure Design Principles

When studying every topic, ask:

- Was security considered during design?
- Are secure defaults used?
- Can this configuration be abused?
- Are dependencies trusted?
- Are secrets protected?
- Can an attacker abuse application logic?

---

# 🐞 Bug Bounty Perspective

Many high-impact bug bounty findings originate from design flaws rather than classic injection vulnerabilities.

Examples include:

- Public cloud storage
- Debug endpoints
- Exposed admin interfaces
- Dependency confusion
- Business logic flaws
- Race conditions
- Misconfigured CORS
- Secret exposure

These issues often require manual testing because automated scanners may not understand business logic or deployment architecture.

---

# 📖 Learning Outcome

After completing this section, you should be able to:

- Identify insecure deployments.
- Recognize risky architectural decisions.
- Assess third-party dependency risks.
- Recommend secure-by-design improvements.
- Apply these concepts during penetration tests and bug bounty engagements.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)