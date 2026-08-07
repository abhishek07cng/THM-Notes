# 🔐 Authentication Methods in Active Directory

> **Module:** Active Directory Basics
>
> Authentication is the process of verifying the identity of a user or computer before access to domain resources is granted. Active Directory primarily uses **Kerberos** for authentication, while **NetNTLM** is maintained for compatibility with older systems.

---

# 📖 Overview

Authentication is one of the core responsibilities of Active Directory.

When a user signs in to a domain, Active Directory verifies their identity before allowing access to resources such as:

- Shared folders
- Printers
- Servers
- Applications

Modern Active Directory environments primarily use **Kerberos**, while **NetNTLM** remains available for legacy systems and compatibility purposes. :contentReference[oaicite:0]{index=0}

---

# 🎯 Learning Objectives

After completing this note, you should understand:

- Why authentication is important
- Kerberos authentication
- Key Distribution Center (KDC)
- Ticket Granting Ticket (TGT)
- Ticket Granting Service (TGS)
- Service Principal Names (SPNs)
- NetNTLM authentication
- Differences between Kerberos and NetNTLM

---

# 🔑 Kerberos Authentication

Kerberos is the **default authentication protocol** used by modern Active Directory domains.

Instead of repeatedly sending a user's password across the network, Kerberos relies on **tickets** that prove a user's identity after successful authentication. :contentReference[oaicite:1]{index=1}

---

# 🏛 Key Distribution Center (KDC)

The **Key Distribution Center (KDC)** runs on the Domain Controller and is responsible for issuing Kerberos tickets.

Its responsibilities include:

- Verifying user credentials
- Issuing authentication tickets
- Issuing service tickets

The KDC is a central component of the Kerberos authentication process. :contentReference[oaicite:2]{index=2}

---

# 🎫 Ticket Granting Ticket (TGT)

When a user successfully logs in:

1. The user sends an authentication request.
2. The KDC verifies the credentials.
3. The KDC issues a **Ticket Granting Ticket (TGT)**.

The TGT proves that the user has already been authenticated and can later be used to request access to other services without sending the password again. :contentReference[oaicite:3]{index=3}

---

# 🎟 Ticket Granting Service (TGS)

When the authenticated user wants to access a service (for example, a file server):

1. The client presents the TGT to the KDC.
2. The KDC verifies the TGT.
3. A **Ticket Granting Service (TGS)** ticket is issued.
4. The client presents the TGS ticket to the requested service.
5. Access is granted if the ticket is valid. :contentReference[oaicite:4]{index=4}

---

# 🏷 Service Principal Name (SPN)

A **Service Principal Name (SPN)** uniquely identifies a service within Active Directory.

Examples include:

- File servers
- SQL Server
- Web servers

The KDC uses the SPN to determine which service the client wants to access and issues the appropriate service ticket. :contentReference[oaicite:5]{index=5}

---

# 🔄 Kerberos Authentication Flow

```text
User Login

↓

Domain Controller (KDC)

↓

Credentials Verified

↓

Ticket Granting Ticket (TGT)

↓

User Requests Service

↓

KDC Issues Service Ticket (TGS)

↓

Client Presents TGS

↓

Service Grants Access
```

---

# 🔐 NetNTLM Authentication

Before Kerberos became the default protocol, Windows used **NTLM**.

Modern Active Directory environments use **NetNTLM** primarily for compatibility with legacy systems that cannot use Kerberos. :contentReference[oaicite:6]{index=6}

---

# ⚙ How NetNTLM Works

NetNTLM uses a **challenge–response** authentication process.

A simplified workflow:

1. The client requests authentication.
2. The server sends a random challenge.
3. The client generates a response using the user's password hash.
4. The server verifies the response.
5. Authentication succeeds or fails.

Because the password itself is not transmitted, the protocol is more secure than sending plaintext credentials, although it is generally considered less secure than Kerberos. :contentReference[oaicite:7]{index=7}

---

# 📊 Kerberos vs NetNTLM

| Kerberos | NetNTLM |
|----------|---------|
| Default authentication protocol | Legacy compatibility protocol |
| Uses tickets | Uses challenge–response |
| Relies on the KDC | Does not rely on Kerberos tickets |
| Better suited for modern AD environments | Mainly used for older systems and applications |

The TryHackMe room presents Kerberos as the primary authentication mechanism and NetNTLM as a compatibility solution for legacy environments. :contentReference[oaicite:8]{index=8}

---

# 📝 TryHackMe Summary

Active Directory primarily authenticates users using **Kerberos**, which relies on a **Key Distribution Center (KDC)** and ticket-based authentication (TGT and TGS). **NetNTLM** remains available for compatibility with older systems and uses a challenge–response mechanism instead of tickets. :contentReference[oaicite:9]{index=9}

---

# 💡 Key Takeaways

- Kerberos is the default authentication protocol in Active Directory.
- The **KDC** running on the Domain Controller issues Kerberos tickets.
- A **TGT** is obtained after successful login.
- A **TGS** is requested when accessing a network service.
- **SPNs** uniquely identify services in the domain.
- **NetNTLM** uses challenge–response authentication and is mainly retained for legacy compatibility.

---

# 📚 References

- TryHackMe – Active Directory Basics