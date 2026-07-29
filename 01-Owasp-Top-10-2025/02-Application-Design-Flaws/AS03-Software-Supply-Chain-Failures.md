# 📦 AS03 - Software Supply Chain Failures

> **Category:** Application Design Flaws
>
> Software Supply Chain Failures occur when an application becomes vulnerable because of insecure third-party software, compromised dependencies, malicious packages, or weaknesses in the software build and deployment pipeline.

---

# 📖 Overview

Modern applications rarely consist entirely of code written by one developer.

Instead they depend on:

- Open-source libraries
- Frameworks
- Package managers
- CI/CD pipelines
- Docker images
- Base operating system images
- Cloud services
- Third-party APIs

Every dependency becomes part of the application's attack surface.

---

# 🎯 Learning Objectives

After studying this topic you should understand:

- What a software supply chain is
- How supply chain attacks happen
- Dependency confusion
- Typosquatting attacks
- Malicious package publishing
- CI/CD compromise
- How to reduce supply chain risk

---

# 🧠 What is a Software Supply Chain?

The software supply chain includes everything required to build, test, package, and deploy an application.

Example:

```
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

Dependencies

↓

Docker Image

↓

Cloud Deployment

↓

Users
```

If any stage is compromised, the final application may also become compromised.

---

# ⚠️ Why Supply Chain Attacks Are Dangerous

Attackers often target trusted components rather than the application itself.

Reasons include:

- One package may be used by millions of applications.
- Developers trust package managers.
- CI/CD systems often have privileged access.
- Updates are installed automatically.

---

# 🔥 Common Supply Chain Risks

## 1️⃣ Vulnerable Dependencies

Applications continue using libraries with publicly known vulnerabilities.

Example:

```
Application

↓

Old Library

↓

Known CVE

↓

Remote Code Execution
```

Always review dependency versions and security advisories.

---

## 2️⃣ Dependency Confusion

Dependency confusion occurs when a package manager downloads an attacker's package instead of the intended internal package.

Example:

Internal package:

```
company-utils
```

Attacker uploads:

```
company-utils
```

to a public registry with a higher version number.

The build system installs the malicious package.

---

## 3️⃣ Typosquatting

Attackers publish packages with names similar to popular libraries.

Example:

```
express
```

↓

```
expres
```

or

```
reqeust
```

instead of

```
request
```

A simple typing mistake installs malicious code.

---

## 4️⃣ Malicious Packages

Some packages intentionally contain:

- Credential stealers
- Cryptocurrency miners
- Remote shells
- Backdoors
- Data exfiltration code

Installing the package immediately compromises the development environment.

---

## 5️⃣ CI/CD Pipeline Compromise

Continuous Integration pipelines often have access to:

- Secrets
- Cloud credentials
- Production deployments
- Signing keys

If compromised, attackers can distribute malicious software to every user.

---

## 6️⃣ Compromised Build Artifacts

Even if source code is secure,

an attacker may modify:

- Build outputs
- Docker images
- Release binaries
- Deployment packages

before distribution.

---

# ⚙️ Attack Flow

```
Attacker

↓

Compromises Package

↓

Developer Installs Dependency

↓

Application Built

↓

Malicious Code Deployed

↓

Users Impacted
```

---

# 🧪 TryHackMe Summary

The TryHackMe room explains that software supply chain failures occur when developers unknowingly trust compromised third-party components.

Examples discussed include:

- Dependency confusion
- Typosquatting
- Malicious packages
- CI/CD compromise

The room emphasises verifying the origin and integrity of dependencies before deployment. :contentReference[oaicite:1]{index=1}

---

# 🌍 Real-World Example

The uploaded TryHackMe material references the **LiteLLM** incident as an example of how a compromised CI environment can result in a malicious package being published to a widely used package registry.

The example illustrates how weaknesses in deployment automation and release controls can affect many downstream users. :contentReference[oaicite:2]{index=2}

---

# 🐞 Bug Bounty Perspective

While supply chain attacks themselves are often outside the scope of traditional web application testing, researchers may identify related issues such as:

- Publicly exposed package registries
- Exposed CI/CD dashboards
- Leaked package tokens
- Public build artifacts
- Exposed Git repositories
- Credential disclosure in build logs

Always follow the target's bug bounty policy before interacting with development infrastructure.

---

# 🔍 Testing Checklist

When reviewing an application or organisation:

- Review dependency versions.
- Check for outdated libraries.
- Look for exposed `.git` repositories.
- Inspect build logs for secrets.
- Identify publicly accessible package registries.
- Check Docker image sources.
- Review CI/CD configuration exposure.

---

# 🛠️ Useful Tools

- npm audit
- pip-audit
- Trivy
- Snyk
- Dependabot
- GitHub Security Advisories
- OWASP Dependency-Check

---

# ❌ Common Developer Mistakes

- Installing untrusted packages.
- Ignoring dependency updates.
- Automatically trusting package registries.
- Hardcoding deployment secrets.
- Running CI pipelines with excessive privileges.
- Not verifying package integrity.

---

# 🛡️ Prevention

- Use trusted package sources.
- Pin dependency versions.
- Regularly update vulnerable libraries.
- Review third-party code before adoption.
- Protect CI/CD credentials.
- Enable package integrity verification.
- Scan dependencies during every build.
- Monitor software supply chain advisories.

---

# ✅ Secure Supply Chain Checklist

- [ ] Dependency inventory maintained.
- [ ] Vulnerability scanning enabled.
- [ ] CI/CD secrets protected.
- [ ] Package versions pinned.
- [ ] Build artifacts verified.
- [ ] Third-party code reviewed.
- [ ] Least privilege applied to build systems.
- [ ] Security monitoring enabled.

---

# 💡 Key Takeaways

- Your application's security depends on the security of its dependencies.
- Package managers should never be trusted blindly.
- CI/CD systems are high-value targets.
- Supply chain attacks can compromise thousands of downstream applications simultaneously.
- Regular dependency review is a core part of secure software development.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Software Supply Chain Guidance