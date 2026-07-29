# ✅ OWASP Web Testing Checklist

Use this checklist during every web application assessment.

---

# Recon

- [ ] Identify technologies
- [ ] Enumerate endpoints
- [ ] Check robots.txt
- [ ] Check sitemap.xml
- [ ] Find hidden directories

---

# Authentication

- [ ] Weak passwords
- [ ] Default credentials
- [ ] MFA testing
- [ ] Password reset
- [ ] Account lockout

---

# Authorization

- [ ] IDOR
- [ ] Vertical privilege escalation
- [ ] Horizontal privilege escalation
- [ ] Forced browsing

---

# Session Management

- [ ] Cookie flags
- [ ] Session fixation
- [ ] Session timeout
- [ ] JWT security

---

# Input Validation

- [ ] SQL Injection
- [ ] XSS
- [ ] SSTI
- [ ] XXE
- [ ] SSRF
- [ ] Command Injection

---

# File Upload

- [ ] MIME bypass
- [ ] Magic bytes
- [ ] Extension bypass
- [ ] SVG upload

---

# APIs

- [ ] Authentication
- [ ] Authorization
- [ ] Mass Assignment
- [ ] Rate limiting

---

# Business Logic

- [ ] Coupon abuse
- [ ] Race conditions
- [ ] Price manipulation
- [ ] Workflow bypass

---

# Security Headers

- [ ] CSP
- [ ] HSTS
- [ ] X-Frame-Options
- [ ] nosniff

---

# Logging

- [ ] Sensitive information disclosure
- [ ] Error messages
- [ ] Stack traces

---

# AI Features

- [ ] Prompt Injection
- [ ] Prompt leakage
- [ ] Tool abuse
- [ ] Data leakage

---

# Final Review

- [ ] Sensitive files
- [ ] Backup files
- [ ] Git exposure
- [ ] Admin panels
- [ ] Debug endpoints