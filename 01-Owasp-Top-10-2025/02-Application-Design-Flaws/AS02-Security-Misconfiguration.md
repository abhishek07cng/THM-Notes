# ⚙️ AS02 - Security Misconfiguration

> **Category:** Application Design Flaws
>
> Security Misconfiguration occurs when applications, frameworks, servers, cloud infrastructure, or deployment environments are configured insecurely, exposing functionality or sensitive information that attackers can abuse.

---

# 📖 Overview

Modern applications rely on numerous components:

- Operating Systems
- Web Servers
- Databases
- Frameworks
- APIs
- Docker Containers
- Kubernetes
- Cloud Services
- Reverse Proxies
- CDNs

Each component requires secure configuration.

A single insecure setting may expose an entire application.

---

# 🎯 Learning Objectives

After studying this topic, you should understand:

- What Security Misconfiguration is
- Why it occurs
- Common configuration mistakes
- How attackers discover them
- Bug bounty testing methodology
- Secure configuration practices

---

# 🧠 What is Security Misconfiguration?

Security Misconfiguration happens when software is deployed with insecure settings.

Examples include:

- Debug mode enabled
- Default passwords
- Directory listing enabled
- Public administration panels
- Exposed cloud storage
- Unnecessary services
- Missing security headers
- Weak CORS configuration

Many attacks require **no exploit**—only finding an insecure configuration.

---

# 🔥 Why It Happens

Common reasons include:

- Default installation settings
- Forgotten development configurations
- Poor deployment procedures
- Lack of hardening
- Inconsistent environments
- Human error
- Insecure cloud defaults

---

# 🔍 Common Examples

## 1️⃣ Default Credentials

```
Username:
admin

Password:
admin
```

or

```
root

password
```

Attackers routinely test well-known default credentials.

---

## 2️⃣ Debug Mode Enabled

Development frameworks often expose debugging interfaces.

Example:

```
Express

NODE_ENV=development
```

Instead of

```
NODE_ENV=production
```

Debug mode may expose:

- Stack traces
- Environment variables
- Database errors
- File paths

---

## 3️⃣ Directory Listing

Instead of serving a web page,

the server exposes:

```
/uploads/

invoice.pdf

users.csv

backup.zip

config.php
```

This allows attackers to browse sensitive files.

---

## 4️⃣ Public Admin Panels

Examples:

```
/admin

/admin/login

/phpmyadmin

/wp-admin

/manage
```

If left exposed, they become high-value attack targets.

---

## 5️⃣ Cloud Storage Exposure

Examples include:

```
Public S3 Bucket

Azure Blob

Google Cloud Storage
```

Misconfigured permissions may expose:

- Backups
- Documents
- Customer data
- Source code

---

## 6️⃣ Container Misconfiguration

Examples:

- Docker socket exposed
- Running as root
- Secrets stored inside images
- Unnecessary ports exposed

---

## 7️⃣ Kubernetes Misconfiguration

Examples:

- Anonymous API access
- Dashboard exposed
- Excessive RBAC permissions
- Public etcd database

---

# ⚙️ Attack Flow

```
Application

↓

Misconfigured Service

↓

Attacker Discovers Exposure

↓

Sensitive Information

↓

Privilege Escalation

↓

System Compromise
```

---

# 🧪 TryHackMe Summary

The TryHackMe room introduces Security Misconfiguration as insecure deployment or operational settings rather than coding errors.

It discusses examples such as:

- Debug configurations
- Default settings
- Directory listing
- Exposed management interfaces
- Cloud and deployment mistakes

These issues can reveal sensitive information or increase the attack surface. :contentReference[oaicite:1]{index=1}

---

# 🌍 Real-World Examples

Examples frequently encountered during security assessments include:

- Public `.git` directories
- Open Jenkins dashboards
- Elasticsearch clusters without authentication
- Public Kubernetes dashboards
- Open Grafana instances
- Public cloud storage buckets
- Exposed backup archives

---

# 🐞 Bug Bounty Methodology

When testing an application, look for:

## Discovery

```
robots.txt

sitemap.xml

security.txt
```

---

## Hidden Directories

```
/admin

/manage

/debug

/backup

/uploads

/test
```

---

## Default Files

```
phpinfo.php

server-status

.env

config.json
```

---

## Information Disclosure

Check whether the application reveals:

- Framework version
- Stack traces
- Absolute file paths
- Database errors
- Internal IP addresses

---

## Cloud Storage

Test for exposed:

- Buckets
- Backup files
- Images
- Logs

---

## Response Headers

Review security headers such as:

- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options
- Referrer-Policy

Missing headers alone are often informational, but combined with another weakness they may increase risk.

---

# 🛠️ Common Testing Tools

- Burp Suite
- ffuf
- dirsearch
- Gobuster
- Nmap
- httpx
- Shodan (where permitted)
- Cloud asset discovery tools

---

# ❌ Common Developer Mistakes

- Deploying development builds to production
- Leaving default passwords unchanged
- Forgetting to disable debug mode
- Exposing backups publicly
- Trusting default cloud permissions
- Running unnecessary services
- Not reviewing production configurations

---

# 🛡️ Prevention

- Disable debug mode in production.
- Remove default credentials.
- Disable directory listing.
- Restrict access to administrative interfaces.
- Apply the Principle of Least Privilege.
- Harden servers before deployment.
- Regularly audit cloud resources.
- Remove unused services and software.
- Protect secrets using dedicated secret management solutions.
- Perform periodic configuration reviews.

---

# ✅ Security Hardening Checklist

- [ ] Production mode enabled
- [ ] Debug disabled
- [ ] Directory listing disabled
- [ ] Default accounts removed
- [ ] Strong authentication enabled
- [ ] Secrets stored securely
- [ ] Admin interfaces restricted
- [ ] Cloud permissions reviewed
- [ ] Security headers configured
- [ ] Logs monitored

---

# 💡 Bug Bounty Tips

When approaching a new target:

1. Identify the technology stack.
2. Search for exposed development resources.
3. Enumerate hidden directories.
4. Review HTTP responses for information leakage.
5. Check for publicly accessible backups.
6. Inspect cloud-hosted assets.
7. Verify whether administrative interfaces are exposed.

Many valuable findings begin with careful observation rather than complex exploitation.

---

# 📝 Key Takeaways

- Security Misconfiguration is usually caused by insecure deployment rather than insecure code.
- Default settings should never be trusted in production.
- Debug information can expose valuable intelligence.
- Cloud infrastructure requires the same security review as application code.
- Regular configuration audits significantly reduce attack surface.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Secure Configuration Guidance