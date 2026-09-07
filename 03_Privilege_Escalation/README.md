# Privilege Escalation

This module collects the supplied TryHackMe privilege-escalation notes and organizes them into a revision-friendly workflow for Linux and Windows.

## Module Structure

```text
03-Privilege-Escalation/
├── 01-Host-Server-Configuration-Reviews.md
├── 02-Linux-Privilege-Escalation-Enumeration.md
├── 03-Linux-Privilege-Escalation-Basics.md
├── 04-Linux-Privilege-Escalation-Automation.md
├── 05-Windows-Privilege-Escalation.md
├── Cheatsheets/
│   ├── Linux-Privilege-Escalation-Cheatsheet.md
│   └── Windows-Privilege-Escalation-Cheatsheet.md
├── Methodology.md
└── Tools.md
```

## Learning Flow

```text
Configuration Review
        ↓
Situational Awareness
        ↓
OS / User / Network / File Enumeration
        ↓
Identify Misconfiguration or Vulnerability
        ↓
Prioritise the Most Direct Path
        ↓
Exploit in Scope
        ↓
Verify Elevated Access
        ↓
Capture Objective / Document the Chain
```

## Core Mental Model

- **Who am I?**
- **What OS and version am I on?**
- **What privileges and groups do I have?**
- **What is running as root/SYSTEM?**
- **What files, scripts, services, and tasks can I modify?**
- **Are credentials exposed?**
- **Are there dangerous privileges or capabilities?**
- **Is there a known vulnerable component?**
- **Can two low-risk findings combine into an escalation path?**

## Topic Progression

1. Host-server configuration reviews and secure baselines
2. Linux manual enumeration
3. Linux privilege-escalation primitives
4. Automated enumeration and public exploits
5. Windows privilege escalation
6. Use the separate cheatsheets for fast revision
