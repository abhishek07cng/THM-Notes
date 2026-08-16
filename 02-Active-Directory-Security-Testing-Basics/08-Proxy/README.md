# 🔐 Proxy CTF — Active Directory Attack Chain

> **Platform:** TryHackMe / CTF  
> **Target:** `10.49.157.149`  
> **Domain:** `ctf.local`  
> **Domain Controller:** `DC01.ctf.local`

This writeup documents the complete attack path captured during the Proxy CTF, from enumeration to credential discovery, NTLMv2 capture/cracking, constrained delegation abuse, Kerberos ticket impersonation, and final remote execution.

## Attack Chain

```text
Port Enumeration
      ↓
Anonymous SMB/RPC Checks
      ↓
IT-Shared Enumeration
      ↓
Discover svc.scanner
      ↓
Writable SMB Share
      ↓
Responder / NTLMv2 Capture
      ↓
NetNTLMv2 Cracking
      ↓
svc.scanner Authentication
      ↓
Constrained Delegation Enumeration
      ↓
S4U / Service Ticket Request
      ↓
Administrator Impersonation
      ↓
Kerberos CCache
      ↓
SMBExec as Administrator
```

## Contents

- [01 — Enumeration & SMB Discovery](01-Enumeration-and-SMB-Discovery.md)
- [02 — Responder & NTLMv2 Capture](02-Responder-and-NTLMv2-Capture.md)
- [03 — Cracking & Credential Validation](03-Cracking-and-Credential-Validation.md)
- [04 — Constrained Delegation & Kerberos Abuse](04-Constrained-Delegation-and-Kerberos-Abuse.md)
- [05 — Final Access & Complete Chain](05-Final-Access-and-Complete-Chain.md)
- [Tools & Commands](Tools.md)

> **Note:** This documentation preserves the commands, observations, failed attempts, and reasoning from the supplied terminal transcript. Credentials and hashes are retained because they are part of the original CTF/lab writeup.
