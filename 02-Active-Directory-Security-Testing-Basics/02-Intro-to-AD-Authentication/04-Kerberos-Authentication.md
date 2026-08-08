# 🎫 Kerberos Authentication

> **TryHackMe Room:** Intro to AD Authentication
>
> **Task:** 4 — Kerberos Authentication

---

# 📖 Overview

Kerberos is a network authentication protocol developed by **MIT** and adopted by Microsoft as the default authentication method starting with **Windows 2000**.

The protocol is named after **Cerberus**, the three-headed dog from Greek mythology that guards the gates of the underworld.

A critical difference between Kerberos and NTLM is **where authentication occurs**.

With NTLM, you authenticate to the service you want to access, which then verifies your identity with the Domain Controller.

With Kerberos, this is reversed:

```text
NTLM:

Client
  ↓
Target Service
  ↓
Domain Controller
  ↓
Authentication Verification
```

```text
Kerberos:

Client
  ↓
Domain Controller
  ↓
Obtain Tickets
  ↓
Target Service
  ↓
Present Ticket
  ↓
Access Granted
```

With Kerberos, you authenticate to the **Domain Controller first** and receive tickets that you then present to services to prove your identity.

---

# 🧩 Key Kerberos Components

Before diving into the authentication flow, it is important to understand the key components.

| Component | Description |
|---|---|
| **Key Distribution Center (KDC)** | A service running on the Domain Controller that handles all ticket requests. It consists of the Authentication Service (AS) and the Ticket Granting Service (TGS). |
| **Authentication Service (AS)** | The component of the KDC that verifies the user's identity and issues the initial Ticket Granting Ticket (TGT). |
| **Ticket Granting Service (TGS)** | The component of the KDC that issues service tickets to users who present a valid TGT. |
| **Ticket Granting Ticket (TGT)** | The initial "primary ticket" issued after successful authentication. This ticket is used to request access to specific services. |
| **Service Ticket (ST)** | A ticket that grants access to a specific service. It is obtained by presenting a TGT to the TGS. |
| **Service Principal Name (SPN)** | A unique identifier for a service instance, used by Kerberos to associate a service with a specific account. |
| **KRBTGT Account** | A special account in AD whose password hash is used to encrypt all TGTs. Compromise of this account allows forging of Golden Tickets. |

---

# 🏛️ Kerberos Components

```text
                    Domain Controller
                           │
                           ▼
                  Key Distribution
                      Center (KDC)
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
       Authentication Service   Ticket Granting Service
               (AS)                     (TGS)
                │                     │
                ▼                     ▼
               TGT                Service Ticket
                                      │
                                      ▼
                               Target Service
```

---

# 🔄 How Kerberos Authentication Works

The Kerberos authentication process involves multiple exchanges between:

- Client
- Key Distribution Center (KDC)
- Target Service

The TryHackMe room divides the process into **5 steps and 8 processes**.

---

# 1️⃣ Step 1 — Authentication Service Request (AS-REQ)

## Process 1 — User Enters Credentials

The user enters their credentials on the client machine.

```text
User
 ↓
Credentials
 ↓
Client Machine
```

---

## Process 2 — AS-REQ

The client sends an **Authentication Service Request (AS-REQ)** to the KDC.

The request contains:

- Username
- Timestamp encrypted with the user's password hash

This is called:

> **Pre-authentication**

```text
Client
   │
   │ AS-REQ
   │ Username + Encrypted Timestamp
   ▼
KDC
```

---

# 2️⃣ Step 2 — Authentication Service Response (AS-REP)

## Process 3 — KDC Verifies Identity

The KDC verifies the user's identity by decrypting the timestamp using the user's password hash stored in Active Directory.

```text
User Password Hash
        +
Encrypted Timestamp
        ↓
Identity Verification
```

---

## Process 4 — AS-REP

If successful, the KDC responds with an **AS-REP** containing:

### Session Key

A session key encrypted with the user's password hash.

### Ticket Granting Ticket

A TGT encrypted with the **KRBTGT account's password hash**.

```text
AS-REP
  │
  ├── Session Key
  │
  └── TGT
```

The client can decrypt the session key, but cannot decrypt or modify the TGT.

Only the KDC can decrypt the TGT.

---

# 3️⃣ Step 3 — Ticket Granting Service Request (TGS-REQ)

When the user wants to access a service, the client sends a **TGS-REQ** to the KDC.

The request contains:

- The TGT received earlier
- The SPN of the service they want to access
- An authenticator containing the username and timestamp encrypted with the session key

```text
Client
   │
   │ TGS-REQ
   │
   ├── TGT
   ├── Service SPN
   └── Authenticator
   │
   ▼
KDC
```

---

# 4️⃣ Step 4 — Ticket Granting Service Response (TGS-REP)

## Process 6 — KDC Validates Request

The KDC decrypts the TGT using the **KRBTGT hash** and validates the request.

If successful, the KDC responds with:

### Service Ticket (ST)

A Service Ticket encrypted with the target service's password hash.

### Service Session Key

A service session key encrypted with the original session key.

```text
TGS-REP
  │
  ├── Service Ticket
  │
  └── Service Session Key
```

---

# 5️⃣ Step 5 — Application Request (AP-REQ)

## Process 7 — Client Presents Service Ticket

The client presents the Service Ticket to the target service.

```text
Client
   │
   │ AP-REQ
   │ Service Ticket
   ▼
Target Service
```

---

## Process 8 — Service Validates Ticket

The service decrypts the ticket using its own password hash.

It validates the user's identity and grants access if the ticket is valid.

```text
Ticket Valid
     ↓
Identity Validated
     ↓
Access Granted
```

---

# 📊 Complete Kerberos Authentication Flow

```text
                         CLIENT
                            │
                            │
                            │ 1. AS-REQ
                            ▼
                  Authentication Service
                            │
                            │ 2. AS-REP
                            │    Session Key + TGT
                            ▼
                         CLIENT
                            │
                            │
                            │ 3. TGS-REQ
                            │    TGT + SPN + Authenticator
                            ▼
                  Ticket Granting Service
                            │
                            │ 4. TGS-REP
                            │    Service Ticket
                            ▼
                         CLIENT
                            │
                            │
                            │ 5. AP-REQ
                            │    Service Ticket
                            ▼
                     TARGET SERVICE
                            │
                            ▼
                       ACCESS GRANTED
```

---

# 🎟️ Ticket Overview

The overall Kerberos authentication process can be simplified as:

```text
Credentials
     ↓
    AS
     ↓
    TGT
     ↓
    TGS
     ↓
Service Ticket
     ↓
Target Service
     ↓
Access
```

---

# ✅ Benefits of Kerberos

Kerberos offers significant advantages over NTLM.

---

## 1. Mutual Authentication

Both the client and server verify each other's identity.

This helps protect against man-in-the-middle attacks.

---

## 2. No Password Transmission

Passwords and hashes are never sent over the network.

Instead, Kerberos uses:

- Encrypted tickets
- Session keys

---

## 3. Single Sign-On (SSO)

Once a user obtains a TGT, they can access multiple services without re-entering their credentials.

```text
Authenticate Once
      ↓
     TGT
      ↓
 ┌────┼────┬────┐
 ↓    ↓    ↓    ↓
SMB  LDAP HTTP MSSQL
```

---

## 4. Delegation Support

Kerberos supports delegation.

This allows services to act on behalf of users to access other resources.

---

## 5. Better Performance

The KDC is contacted during:

- Initial authentication
- Requests for new service tickets

Services can validate tickets locally without contacting the Domain Controller for every request.

---

## 6. Time-Limited Tickets

Kerberos tickets have configurable lifetimes.

The TryHackMe room notes that TGTs typically have a lifetime of approximately **10 hours**.

Time-limited tickets restrict the window of opportunity available to an attacker using a stolen ticket.

---

# ❌ Drawbacks of Kerberos

Despite its improvements over NTLM, Kerberos has its own weaknesses and requirements.

---

## 1. Time Synchronisation Required

Kerberos requires clocks to be synchronized within approximately **5 minutes**.

Large differences in system time can cause authentication failures.

```text
Client Time
     ↕
Must be closely synchronized
     ↕
Domain Controller Time
```

---

## 2. Single Point of Failure

The KDC is critical to Kerberos authentication.

If the KDC becomes unavailable:

```text
KDC Unavailable
      ↓
Kerberos Authentication Fails
```

NTLM may be used as a fallback in some situations.

---

## 3. Vulnerable to Ticket Attacks

Stolen Kerberos tickets can potentially be used to impersonate users.

Examples include:

- Pass-the-Ticket
- Pass-the-ccache

A compromised **KRBTGT hash** can also allow attackers to forge **Golden Tickets**.

---

## 4. Kerberoasting

Service tickets are encrypted using service account password hashes.

Authenticated users can request service tickets for accounts with registered SPNs.

These tickets can then potentially be cracked offline if weak service account passwords are used.

---

## 5. Configuration Complexity

Kerberos requires correct configuration of:

- SPNs
- DNS
- Time synchronization

Incorrect configuration can cause authentication problems and make Kerberos more complex to deploy and troubleshoot.

---

# 📁 Credential Cache (ccache) Files

On Linux systems, Kerberos stores tickets in **credential cache files**, commonly called:

> **ccache files**

These files can contain:

- The user's TGT
- Service Tickets obtained during the session

---

# 📍 Default ccache Location

The TryHackMe room gives the following default location:

```text
/tmp/krb5cc_%{uid}
```

For example:

```text
/tmp/krb5cc_1000
```

---

# 🌐 KRB5CCNAME

The:

```text
KRB5CCNAME
```

environment variable specifies which ccache file should be used.

### Example

```bash
export KRB5CCNAME=mary.ccache
```

---

# 🔎 Viewing Kerberos Tickets

The:

```bash
klist
```

command displays tickets stored in the current credential cache.

---

# ⚠️ Security Importance of ccache Files

Tools such as **Impacket** can use ccache files to authenticate without requiring the user's password.

If an attacker obtains a user's ccache file, they may be able to authenticate as that user without knowing their password.

This is associated with attacks such as:

- Pass-the-Ticket
- Pass-the-ccache

---

# 🧪 Practical Demonstration — Kerberos Authentication

> **TryHackMe Lab**

The room demonstrates Kerberos authentication using **Impacket**.

The demonstration follows this process:

```text
Obtain TGT
   ↓
Store TGT in ccache
   ↓
Set KRB5CCNAME
   ↓
Use ccache for authentication
   ↓
Request Service Ticket
   ↓
Authenticate to SMB
```

---

# 💻 Step 1 — Add SERVER1 to `/etc/hosts`

Because Kerberos relies on DNS and SPNs, the room first hardcodes the `SERVER1` hostname.

```bash
echo 192.168.11.51 SERVER1.thm.loc >> /etc/hosts
```

This creates the following local mapping:

```text
192.168.11.51 → SERVER1.thm.loc
```

---

# 💻 Step 2 — Request a TGT

The room uses `getTGT.py` from Impacket to request a Ticket Granting Ticket.

```bash
getTGT.py thm.loc/mary:'SuperLongForKerberos123!' -dc-ip 192.168.11.100
```

### Example Output

```text
Impacket v0.10.1.dev1+20230316.112532.f0ac44bd - Copyright 2022 Fortra

[*] Saving ticket in mary.ccache
```

This creates a ccache file named:

```text
mary.ccache
```

in the current directory.

---

# 🔍 Command Breakdown — `getTGT.py`

```bash
getTGT.py thm.loc/mary:'SuperLongForKerberos123!' -dc-ip 192.168.11.100
```

### `getTGT.py`

An Impacket tool used to request a Kerberos **Ticket Granting Ticket (TGT)**.

### `thm.loc/mary`

Specifies:

```text
Domain: thm.loc
User:   mary
```

### `'SuperLongForKerberos123!'`

The password used by the TryHackMe lab account.

### `-dc-ip`

Specifies the IP address of the Domain Controller:

```text
192.168.11.100
```

---

# 💻 Step 3 — Set KRB5CCNAME

The room then sets the `KRB5CCNAME` environment variable to point to the newly created ccache file.

```bash
export KRB5CCNAME=mary.ccache
```

This tells Kerberos-aware tools which credential cache to use.

---

# 💻 Step 4 — Authenticate to SMB Using Kerberos

The room then connects to the SMB share using Kerberos authentication:

```bash
smbclient.py thm.loc/mary@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

### Example Output

```text
Impacket v0.10.1.dev1+20230316.112532.f0ac44bd - Copyright 2022 Fortra

Type help for list of commands
#
```

---

# 🔍 Command Breakdown — `smbclient.py`

### `-k`

Tells the tool to use **Kerberos authentication**.

### `-no-pass`

Indicates that authentication should use the existing Kerberos ticket instead of a password.

### `-dc-ip 192.168.11.100`

Specifies where the Domain Controller / KDC can be found.

### `SERVER1.thm.loc`

The hostname of the target server.

---

# ⚠️ Important Kerberos Note

When using Kerberos, you must use the **hostname rather than the IP address**.

Kerberos relies on **Service Principal Names (SPNs)**, which are tied to DNS names.

Therefore:

### Correct

```text
SERVER1.thm.loc
```

### Instead of

```text
192.168.11.51
```

---

# ⚙️ Behind the Scenes

The TryHackMe room describes the following process:

```text
Client
   │
   │ TGT from ccache
   ▼
KDC
   │
   │ Service Ticket
   ▼
Client
   │
   │ Service Ticket
   ▼
Target SMB Server
   │
   │ Ticket Validation
   ▼
Access Granted
```

More specifically:

1. The client uses the TGT from the ccache file to request a Service Ticket from the KDC.
2. The KDC issues a Service Ticket for the SMB service.
3. The client presents the Service Ticket to the target host.
4. The target host validates the ticket.
5. Access is granted.

---

# 🧠 Complete Practical Flow

```text
User Credentials
       ↓
getTGT.py
       ↓
Ticket Granting Ticket
       ↓
mary.ccache
       ↓
KRB5CCNAME
       ↓
smbclient.py -k -no-pass
       ↓
TGS Request
       ↓
Service Ticket
       ↓
SMB Server
       ↓
Access Granted
```

---

# 🎯 NTLM vs Kerberos

| Feature | NTLM | Kerberos |
|---|---|---|
| Authentication model | Challenge-response | Ticket-based |
| Initial authentication | Through target service | Through KDC |
| Mutual authentication | ❌ No | ✅ Yes |
| Password transmission | Password itself is not sent | Password/hash is not sent over network |
| Single Sign-On | Limited | ✅ Yes |
| Uses tickets | ❌ | ✅ |
| Requires time synchronization | ❌ | ✅ |
| Uses SPNs | ❌ | ✅ |
| Major attack examples | Pass-the-Hash, NTLM Relay | Pass-the-Ticket, Kerberoasting, Golden Ticket |

---

# 💡 Key Takeaways

- Kerberos is a network authentication protocol developed by MIT.
- Microsoft adopted Kerberos as the default authentication method starting with Windows 2000.
- Kerberos uses a ticket-based authentication system.
- The KDC runs on the Domain Controller.
- The KDC consists of the Authentication Service (AS) and Ticket Granting Service (TGS).
- A successful initial authentication results in a TGT.
- A TGT is used to request Service Tickets.
- SPNs identify specific services.
- The KRBTGT account is responsible for protecting TGTs.
- Kerberos provides mutual authentication.
- Kerberos supports Single Sign-On.
- Kerberos requires synchronized system clocks.
- Kerberos tickets are stored in credential caches on Linux systems.
- `KRB5CCNAME` specifies the credential cache being used.
- `klist` can display tickets stored in the current ccache.
- Impacket can use Kerberos tickets for authentication.
- Kerberos authentication requires the correct hostname because SPNs are tied to DNS names.
- Stolen tickets can be abused through attacks such as Pass-the-Ticket.
- Compromise of the KRBTGT hash can enable Golden Ticket attacks.

---

# 📚 References

- TryHackMe — Intro to AD Authentication