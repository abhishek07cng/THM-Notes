# 📦 A08 – Software and Data Integrity Failures

> **Category:** Insecure Data Handling
>
> Software or Data Integrity Failures occur when an application trusts software, updates, configuration files, or data without verifying that they are authentic, unmodified, and from a trusted source.

---

# 📖 Overview

Modern applications constantly rely on external components:

- Software updates
- Open-source packages
- Configuration files
- CI/CD pipelines
- Templates
- JSON data
- Third-party libraries
- AI models

If these resources are accepted without verification, attackers may introduce malicious code or manipulated data.

---

# 🎯 Learning Objectives

After studying this topic you should understand:

- What Software or Data Integrity Failures are
- Why integrity verification matters
- Trust boundaries
- Secure software updates
- CI/CD security
- AI supply-chain risks
- Prevention techniques

---

# 🧠 What are Software or Data Integrity Failures?

Software or Data Integrity Failures occur when an application assumes software or data is trustworthy without verifying:

- Authenticity
- Integrity
- Origin

Examples include:

- Installing unsigned updates
- Loading scripts from untrusted sources
- Trusting modified configuration files
- Accepting altered JSON data
- Running unverified binaries

The application assumes the content is safe when that assumption has never been verified. :contentReference[oaicite:2]{index=2}

---

# ⚠️ Why It Matters

Applications increasingly depend on external software and data.

Attackers who compromise these trusted resources can:

- Execute malicious code
- Steal credentials
- Deploy backdoors
- Compromise entire organisations

Supply-chain attacks demonstrate that trusted software can become an attack vector if integrity is not verified.

---

# 🔍 Common Integrity Failures

## 1️⃣ Unverified Software Updates

Installing updates without verifying:

- Digital signatures
- Checksums
- Publisher authenticity

may allow malicious software to be installed.

---

## 2️⃣ Trusting Untrusted Sources

Examples include:

- External scripts
- Templates
- Configuration files
- JSON data
- Plugins

Applications should verify that these resources originate from trusted sources before using them. :contentReference[oaicite:3]{index=3}

---

## 3️⃣ Insecure CI/CD Pipelines

Modern software is often automatically built and deployed.

If attackers compromise:

- Build servers
- CI workflows
- Package publishing credentials

they may distribute malicious software through legitimate release channels.

---

## 4️⃣ Dependency Supply-Chain Attacks

Applications frequently rely on third-party packages.

Risks include:

- Compromised packages
- Dependency confusion
- Typosquatting
- Malicious updates

Developers should verify package integrity and carefully manage dependencies.

---

## 5️⃣ AI Model Integrity

The TryHackMe room extends the same trust problem to AI.

Examples include:

- Downloading unverified models
- Using modified datasets
- Loading tampered model weights

An attacker may introduce hidden behaviour into a model that activates only under specific trigger inputs, making integrity verification especially important. :contentReference[oaicite:4]{index=4}

---

# 🔐 Establish Trust Boundaries

The room explains that preventing these failures begins by establishing **trust boundaries**.

Before executing software or accepting data, ask:

- Who created it?
- Has it been modified?
- Can its integrity be verified?
- Is it from a trusted source?

Never assume external content is trustworthy simply because it appears legitimate. :contentReference[oaicite:5]{index=5}

---

# ⚙️ Attack Flow

```text
Application

↓

Downloads Software / Data

↓

No Integrity Verification

↓

Attacker Modifies Resource

↓

Application Executes Malicious Content
```

---

# 🧪 TryHackMe Summary

The TryHackMe room highlights that Software or Data Integrity Failures occur when applications trust code, updates, or data without verifying authenticity or integrity.

Examples include:

- Unsafely trusted updates
- Unverified configuration files
- Modified JSON or templates
- Compromised dependencies
- Weak CI/CD security

It recommends verifying software before execution and enforcing clear trust boundaries. :contentReference[oaicite:6]{index=6}

---

# 🌍 Real-World Examples

Examples commonly encountered include:

- Installing unsigned software updates
- Pulling dependencies using unpinned versions
- Automatically deploying unreviewed code
- Running scripts from unknown sources
- Loading modified AI models

---

# 🤖 AI Security Perspective

The uploaded material highlights that integrity concerns now extend to AI systems.

Potential risks include:

- Backdoored model weights
- Tampered training datasets
- Automatically trusted AI components

These issues reinforce the need to verify the origin and integrity of AI assets before deployment. :contentReference[oaicite:7]{index=7}

---

# 🐞 Bug Bounty Perspective

Integrity issues often appear as:

- Weak CI/CD controls
- Unsafely trusted package updates
- Missing signature verification
- Build pipeline weaknesses
- Unverified dependency downloads

These findings can have significant impact because they affect every user of the compromised software.

---

# 🛡️ How to Prevent Software & Data Integrity Failures

The TryHackMe room recommends:

- Establish clear trust boundaries.
- Verify software authenticity before execution.
- Validate integrity using signatures or checksums.
- Download software only from trusted sources.
- Protect CI/CD pipelines.
- Verify packages before deployment.
- Avoid automatically trusting external data. :contentReference[oaicite:8]{index=8}

---

# 🔒 Secure Development Checklist

- [ ] Software signatures verified.
- [ ] Checksums validated.
- [ ] Trusted package sources used.
- [ ] CI/CD pipeline secured.
- [ ] Dependencies reviewed.
- [ ] Automatic deployments protected.
- [ ] External data verified.
- [ ] Trust boundaries documented.

---

# ❌ Common Developer Mistakes

- Automatically trusting downloaded software.
- Using unverified dependencies.
- Ignoring package signatures.
- Allowing unrestricted CI/CD publishing.
- Trusting external configuration files.
- Loading AI models from unknown sources.
- Executing software without integrity verification.

---

# 💡 Key Takeaways

- Integrity means ensuring software and data have not been modified.
- Never trust software simply because it is widely used.
- Verify updates, packages, and configuration files before use.
- Protect CI/CD pipelines against unauthorised changes.
- Modern AI systems require the same integrity protections as traditional software.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)