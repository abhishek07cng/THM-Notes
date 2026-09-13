# Metasploit

A structured collection of notes, commands, workflows, and practical knowledge covering the **Metasploit Framework**, from the fundamentals of `msfconsole` to scanning, exploitation, post-exploitation, payload generation, weaponisation, shells, listeners, and payload delivery.

> **Purpose:** Practical cybersecurity learning and authorized security testing.

---

## 📚 Modules

| # | Module | Topics Covered |
|---|---|---|
| 01 | [Metasploit: The Basics](01-Metasploit-The-Basics.md) | Metasploit Framework, `msfconsole`, modules, exploits, payloads, options, sessions, basic workflows |
| 02 | [Metasploit: Scanning and Exploitation](02-Metasploit-Scanning-and-Exploitation.md) | Scanning, enumeration, service discovery, exploit selection, and exploitation workflows |
| 03 | [Metasploit: Post-Exploitation](03-Metasploit-Post-Exploitation.md) | Sessions, system information, privilege escalation concepts, credential collection, and post-exploitation |
| 04 | [Metasploit: Payload Generation](04-Metasploit-Payload-Generation.md) | Payload concepts, payload generation, handlers, staged/stageless payloads, and delivery |
| 05 | [Exploitation and Weaponisation](05-Exploitation-and-Weaponisation.md) | Exploitation concepts, weaponisation techniques, payload preparation, and attack workflows |
| 06 | [Shells & Listeners Fundamentals](06-Shells-and-Listeners-Fundamentals.md) | Bind shells, reverse shells, listeners, connections, and shell fundamentals |
| 07 | [Shell Payload Generation & Delivery](07-Shell-Payload-Generation-and-Delivery.md) | Shell payload creation, delivery methods, listeners, and practical payload workflows |

---

## 🗺️ Learning Path

```text
┌──────────────────────────────┐
│ 01. Metasploit: The Basics   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 02. Scanning & Exploitation  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 03. Post-Exploitation        │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 04. Payload Generation       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 05. Exploitation &           │
│     Weaponisation            │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 06. Shells & Listeners       │
│     Fundamentals             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 07. Shell Payload Generation │
│     & Delivery               │
└──────────────────────────────┘
```

---

## 🎯 What This Collection Covers

### Metasploit Fundamentals
- Metasploit Framework architecture
- `msfconsole`
- Modules and module categories
- Vulnerabilities, exploits, and payloads
- Searching for modules
- Reading module information
- Configuring module options
- Selecting payloads
- Running exploits
- Checking exploitability
- Managing sessions

### Scanning & Exploitation
- Target discovery
- Service and port enumeration
- Identifying vulnerable services
- Searching for suitable exploits
- Configuring exploitation modules
- Executing authorized exploits

### Post-Exploitation
- Managing compromised sessions
- Gathering system information
- Working with Meterpreter/shell sessions
- Post-exploitation workflows
- Maintaining awareness of the compromised environment

### Payloads
- Payload fundamentals
- Staged and stageless payloads
- Payload generation
- Payload configuration
- Handlers and listeners
- Payload delivery concepts

### Shells
- Shell fundamentals
- Bind shells
- Reverse shells
- Listeners
- Shell payloads
- Establishing and managing connections

---

## 🛠️ Core Tools & Concepts

The modules primarily focus on:

```text
Metasploit Framework
        │
        ├── msfconsole
        ├── Exploit Modules
        ├── Auxiliary Modules
        ├── Payloads
        ├── Post-Exploitation Modules
        ├── Encoders
        ├── NOPs
        └── Evasion
```

Supporting concepts include:

- Vulnerability assessment
- Enumeration
- Exploitation
- Payloads
- Sessions
- Shells
- Listeners
- Post-exploitation
- Payload delivery

---

## 📂 Repository Structure

```text
Metasploit/
│
├── README.md
│
├── 01-Metasploit-The-Basics.md
├── 02-Metasploit-Scanning-and-Exploitation.md
├── 03-Metasploit-Post-Exploitation.md
├── 04-Metasploit-Payload-Generation.md
├── 05-Exploitation-and-Weaponisation.md
├── 06-Shells-and-Listeners-Fundamentals.md
└── 07-Shell-Payload-Generation-and-Delivery.md
```

Each module is maintained as an individual Markdown file so it can be studied independently and used as a practical reference during authorized labs and CTFs.

---

## 🔄 General Metasploit Workflow

A typical workflow covered throughout these modules can be represented as:

```text
Reconnaissance
      │
      ▼
Scanning & Enumeration
      │
      ▼
Identify Vulnerability
      │
      ▼
Search Metasploit
      │
      ▼
Select Exploit
      │
      ▼
Configure Options
      │
      ▼
Select Payload
      │
      ▼
Check / Run Exploit
      │
      ▼
Obtain Session
      │
      ▼
Post-Exploitation
```

The exact workflow depends on the target, vulnerability, exploit, payload, and testing objective.

---

## 📌 Practical Reference

The individual module files are designed to serve two purposes:

**Learning**
- Understand how Metasploit works.
- Learn the purpose of different module types.
- Understand exploitation and payload concepts.
- Build familiarity with shells, listeners, and sessions.

**Practical Use**
- Quickly recall Metasploit commands.
- Review exploitation workflows.
- Reference payload and listener concepts.
- Use during authorized CTFs, labs, and penetration-testing exercises.

---

## ⚠️ Authorization & Safety

Metasploit is a powerful penetration-testing framework. The techniques and commands in this repository should only be used against:

- Systems you own.
- Intentionally vulnerable lab environments.
- CTF machines.
- Systems for which you have explicit authorization to test.

Do **not** use these techniques against unauthorized systems, networks, accounts, or services.

---

## 📈 Progress

| Module | Status |
|---|---|
| 01 — Metasploit: The Basics | ✅ Completed |
| 02 — Metasploit: Scanning and Exploitation | ⬜ |
| 03 — Metasploit: Post-Exploitation | ⬜ |
| 04 — Metasploit: Payload Generation | ⬜ |
| 05 — Exploitation and Weaponisation | ⬜ |
| 06 — Shells & Listeners Fundamentals | ⬜ |
| 07 — Shell Payload Generation & Delivery | ⬜ |

---

## 🔗 Quick Navigation

**Start here → [01 — Metasploit: The Basics](01-Metasploit-The-Basics.md)**

Then continue through the modules in order to build from Metasploit fundamentals toward practical exploitation, payload, shell, and delivery workflows.
