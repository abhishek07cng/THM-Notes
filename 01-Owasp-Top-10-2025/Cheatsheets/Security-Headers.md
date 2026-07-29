# 🛡 Security Headers Cheat Sheet

Security headers help protect web applications against common attacks.

---

# Content-Security-Policy (CSP)

Controls which resources the browser may load.

Example

```
Content-Security-Policy:
default-src 'self'
```

Protects against:

- XSS
- Data Injection

---

# Strict-Transport-Security (HSTS)

```
Strict-Transport-Security:
max-age=31536000
```

Forces HTTPS.

Protects against:

- SSL stripping

---

# X-Frame-Options

```
DENY

SAMEORIGIN
```

Protects against:

- Clickjacking

---

# X-Content-Type-Options

```
nosniff
```

Stops MIME sniffing.

---

# Referrer-Policy

Controls referrer information.

Example

```
strict-origin-when-cross-origin
```

---

# Permissions-Policy

Restricts browser APIs.

Example

```
camera=()
microphone=()
geolocation=()
```

---

# Cross-Origin-Resource-Policy

Controls resource sharing.

---

# Cross-Origin-Opener-Policy

Helps isolate browsing contexts.

---

# Cross-Origin-Embedder-Policy

Required for secure cross-origin isolation.

---

# During Pentesting

Check:

- Missing CSP
- Weak CSP
- Missing HSTS
- Missing X-Frame-Options
- Missing nosniff
- Unsafe Referrer Policy

---

# Burp Suite

Proxy

↓

Response Headers

↓

Check Security Headers

---

# References

OWASP Secure Headers Project