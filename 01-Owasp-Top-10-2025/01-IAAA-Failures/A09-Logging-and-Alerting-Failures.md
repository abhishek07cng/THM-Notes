# 📋 A09 - Logging and Alerting Failures

> **Category:** IAAA Failures
>
> Logging and Alerting Failures occur when an application fails to record, monitor, or alert on security-relevant events. Without proper logging, organizations cannot effectively detect, investigate, or respond to security incidents.

---

# 📖 Overview

Modern applications constantly generate security events such as:

- User logins
- Failed authentication attempts
- Password changes
- Privilege changes
- File access
- API requests
- Administrative actions

These events should be securely recorded and monitored.

If important events are not logged—or if no alerts are generated for suspicious activity—attackers can operate without being detected.

---

# 🎯 Objectives

After studying this topic, you should understand:

- What Logging & Alerting Failures are
- Why security logging is important
- Common logging failures
- How attackers exploit poor logging
- Investigation best practices
- Prevention techniques

---

# 🧠 What is Logging?

Logging is the process of recording events that occur within an application or system.

Security logs provide a historical record of user activities, system behaviour, and security events.

They help answer questions such as:

- Who performed an action?
- What action occurred?
- When did it happen?
- Where did it originate?
- Was the action successful?

---

# 🚨 What is Alerting?

Alerting is the process of notifying administrators when suspicious or high-risk activities occur.

Examples include:

- Multiple failed login attempts
- Privilege escalation
- Password changes
- Large data exports
- Suspicious API usage
- Unexpected administrator activity

Without alerting, attacks may go unnoticed even if they are recorded in logs.

---

# ⚠️ Common Logging & Alerting Failures

According to the TryHackMe room, common failures include:

- Missing authentication logs
- Missing privilege change logs
- Poor error messages
- Missing alerts
- Short log retention periods
- Logs stored where attackers can modify or delete them

These failures reduce visibility during security incidents. :contentReference[oaicite:0]{index=0}

---

# 🔍 Examples

## Missing Failed Login Logs

```
Attacker

↓

Attempts

↓

password1

↓

password2

↓

password3

↓

No Logs Generated
```

Security teams have no visibility into the brute-force attack.

---

## Missing Privilege Change Logs

```
User

↓

Role changed

↓

Admin

↓

No audit record
```

Investigators cannot determine when or how privileges changed.

---

## Poor Error Logging

Instead of:

```
User "alice"

Failed login

Incorrect password

Source IP:
203.0.113.15
```

The application records only:

```
Error occurred
```

This provides little value during incident response.

---

## Missing Alerts

Imagine:

```
10,000

Failed Logins

↓

No Notification

↓

Administrator unaware
```

The attack continues undetected.

---

## Log Tampering

If logs are stored on the same compromised server with weak permissions:

```
Attacker

↓

Deletes logs

↓

No forensic evidence
```

This makes post-incident investigation much more difficult.

---

# ⚙️ Investigation Workflow

```
Security Event

↓

Log Generated

↓

Stored Securely

↓

Monitored

↓

Alert Triggered

↓

Security Team Investigates

↓

Incident Response
```

Every stage is important.

---

# 🧪 TryHackMe Lab Summary

The TryHackMe exercise demonstrates how security investigators rely on application logs to reconstruct an attack.

It also encourages considering how difficult an investigation would become if critical log information were missing.

The room highlights that accountability depends on reliable logging and alerting. :contentReference[oaicite:1]{index=1}

---

# 🌍 Real-World Perspective

The uploaded material identifies several practical issues:

- Brute-force attacks become invisible without failed login logs.
- Privilege escalation leaves no evidence if role changes are not recorded.
- Generic log messages hinder investigations.
- Logs stored on compromised hosts are vulnerable to deletion or modification.
- Short retention periods may remove evidence before an incident is discovered. :contentReference[oaicite:2]{index=2}

---

# 🐞 Bug Bounty Perspective

Although logging itself is usually an internal control, bug bounty hunters may identify related weaknesses such as:

- Missing account lockout despite repeated login attempts.
- No notifications for sensitive account changes.
- Ability to delete or manipulate audit records.
- Administrative actions that leave no audit trail.

These issues can increase the impact of other vulnerabilities.

---

# 🔍 Logging Checklist

When reviewing an application, consider:

- Are failed logins recorded?
- Are successful logins recorded?
- Are password changes logged?
- Are privilege changes logged?
- Are account lockouts logged?
- Are administrative actions audited?
- Are security alerts generated?
- Are logs protected against tampering?
- Is log retention appropriate?

---

# ❌ Common Developer Mistakes

- Logging too little information.
- Logging sensitive information such as passwords.
- Using vague error messages.
- Storing logs on the same server without protection.
- Not monitoring security logs.
- No alerting for suspicious behaviour.
- Short log retention periods.

---

# 🛡️ Prevention

- Log all authentication events.
- Record privilege changes.
- Protect log integrity.
- Store logs centrally where possible.
- Monitor logs continuously.
- Configure alerts for high-risk activities.
- Define appropriate log retention policies.
- Regularly review audit logs.

---

# 📝 Key Takeaways

- Logging provides accountability.
- Alerting enables rapid detection of attacks.
- Missing logs make incident investigations difficult.
- Protected and monitored logs improve security visibility.
- Logging and alerting are essential components of a secure application.

---

# 📚 References

- TryHackMe – OWASP Top 10 (2025)
- OWASP Logging Cheat Sheet