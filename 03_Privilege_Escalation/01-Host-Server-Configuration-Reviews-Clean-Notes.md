# Host-Server Configuration Reviews

> **Source:** Supplied THM material  
> **Purpose:** Understand host configuration weaknesses that can lead to privilege escalation and build a repeatable LPE enumeration methodology.

---

## ⚡ Quick Revision

### Core Idea

A host can be **fully patched** and still be vulnerable to privilege escalation if its configuration deviates from a secure baseline.

### 🔐 Two Privilege-Escalation Categories

| Type | Targets | Examples |
|---|---|---|
| **Vulnerability-Based** | Software/code flaws | Kernel vulnerabilities, buffer overflows, known CVEs |
| **Configuration-Based** | System/admin configuration | Weak permissions, insecure services, exposed credentials |

### 🎯 Six Misconfiguration Categories

```text
1. Users & Groups
2. File & Directory Permissions
3. Services
4. Scheduled Tasks / Cron
5. Credential Storage
6. Network Configuration
```

### 🧭 3-Phase LPE Methodology

```text
┌─────────────────────────────────────┐
│  PHASE 1 — SITUATIONAL AWARENESS    │
│  Who am I? What system is this?     │
└──────────────────┬──────────────────┘
                   ↓
┌─────────────────────────────────────┐
│  PHASE 2 — CATEGORY ENUMERATION     │
│  Users → Files → Services → Tasks   │
│  → Credentials → Network            │
└──────────────────┬──────────────────┘
                   ↓
┌─────────────────────────────────────┐
│  PHASE 3 — PRIORITISE & CHAIN        │
│  Direct → Chained → Hardening        │
└──────────────────┬──────────────────┘
                   ↓
             ELEVATED ACCESS
```

### 🚦 Finding Priority

| Priority | Meaning | Example |
|---|---|---|
| 🔴 **High** | Direct escalation | Exposed root/admin credentials |
| 🟠 **Medium** | Requires chaining | Writable script executed by privileged cron |
| 🟢 **Low** | Defence-in-depth | Hardening weakness with no clear LPE path |

---

## 📌 Table of Contents

- [1. Introduction](#1-introduction)
- [2. Configuration Review](#2-configuration-review)
- [3. Security Baselines & Frameworks](#3-security-baselines--frameworks)
- [4. Automated Compliance Tooling](#4-automated-compliance-tooling)
- [5. Categories of Misconfiguration](#5-categories-of-misconfiguration)
- [6. Structured Enumeration Methodology](#6-structured-enumeration-methodology)
- [7. Reading a CIS Benchmark](#7-reading-a-cis-benchmark)
- [Final Takeaways](#final-takeaways)

---

# Host-Server Configuration Reviews**

> Source-derived notes from the supplied THM material. Commands, examples, and lab details are retained where relevant.

## Host-Server Configuration Reviews**

## 1. Introduction

> 💡 **Key Point:** **Privilege escalation** usually happens after initial access, when a tester looks for a path from a low-privileged account to higher privileges.
**

Every operating system ships with a set of default configurations. Administrators then layer their own changes on top, installing software, creating user accounts, configuring services, and setting permissions. When those configurations deviate from secure practices, whether through oversight, convenience, or a lack of awareness, they introduce weaknesses that can be exploited to escalate privileges on the host. This room provides the conceptual foundation for identifying and understanding those weaknesses.

**Privilege escalation** is the process of moving from a lower-privileged account to a higher-privileged one on a system. It is a critical phase of almost every penetration test. An attacker who gains initial access to a host, typically through a web application vulnerability, phishing, or a compromised service, will rarely land with administrative or root-level access. The next step is to find a way to elevate privileges, and misconfigured hosts are one of the most reliable sources of opportunity.

There are broadly two categories of privilege escalation. The first is vulnerability-based escalation, which exploits bugs in software, such as kernel vulnerabilities, buffer overflows in installed applications, or known CVEs in running services. The second is configuration-based escalation, which exploits errors in how the system has been set up, such as overly permissive file permissions, insecure service configurations, or plaintext credentials left in accessible locations.

This room focuses on the second category. A fully patched system with no known software vulnerabilities can still be trivially escalated if its configuration is poor. In practice, configuration-based escalation is often more common and more reliable than vulnerability-based escalation, particularly in mature environments where patching is consistent but hardening is not.

The room does not cover specific exploitation techniques. Instead, it establishes the knowledge you need before approaching a host for privilege escalation. You will learn what a secure configuration looks like according to industry-recognised frameworks, what categories of misconfiguration commonly lead to escalation, and how to approach a new host with a structured methodology. The rooms that follow in this module apply these concepts hands-on to Linux and Windows systems.

### Learning Objectives**

By the end of this room, you will be able to:

Explain what a host-server configuration review is and when it is performed during a penetration testing engagement.

Distinguish between vulnerability-based privilege escalation and configuration-based privilege escalation.

Describe the purpose and structure of **CIS Benchmarks** and **DISA STIGs**, and how they define a secure configuration baseline.

Identify the role of automated compliance auditing tools such as **Nessus**, **Lynis**, and **OpenSCAP** in performing configuration reviews at scale.

Identify the major categories of host misconfiguration that lead to privilege escalation, including file permissions, service configurations, scheduled tasks, and credential storage.

Apply a structured enumeration methodology that maps to benchmark categories when approaching a new host.

## 2. Configuration Review**

A configuration review is a structured audit of a host's settings, permissions, services, and policies, measured against an accepted secure baseline. Its purpose is to identify deviations from that baseline that could introduce security risk. In the context of penetration testing, those deviations are potential vectors for privilege escalation.

Configuration reviews differ from vulnerability scanning. A vulnerability scanner such as **Nessus** or Qualys identifies known software flaws, missing patches, and CVEs. A configuration review, by contrast, focuses on how the system is set up rather than what software it runs. A host can pass a vulnerability scan with no findings and still fail a configuration review because its services are running with excessive privileges, its file permissions are too broad, or its administrators have left credentials in accessible locations.

Configuration reviews also differ from exploit development. An exploit targets a specific software bug with a crafted payload. A configuration review identifies weaknesses in the way a host is administered. The skills are complementary, but the mindset is different. Exploitation asks "what is broken in this software?" Configuration review asks "what is set up incorrectly on this system?"
### When Configuration Reviews Happen
During a penetration test, configuration reviews typically occur during the post-exploitation phase, after initial access to a host has been obtained. Once a tester has a shell on a target system, the next objective is usually to escalate privileges. The configuration review is the systematic process of examining the host for misconfigurations that enable that escalation.

Configuration reviews can also be conducted as standalone assessments, independent of a full penetration test. In this context, a security team or auditor is given authenticated access to hosts and evaluates their configuration against a defined baseline. The findings are reported as hardening gaps rather than exploitable vulnerabilities, but the checks performed are the same.
### The Offensive Perspective
From a defensive standpoint, configuration reviews are a compliance and hardening exercise. Security teams audit hosts to ensure they meet organisational standards, regulatory requirements, and industry best practices. The goal is remediation.

From an offensive standpoint, the process is identical but the goal is reversed. A penetration tester auditing a host's configuration is looking for the same deviations a compliance auditor would flag, but with the intent to exploit them rather than fix them. The same checklist that a system administrator uses to harden a server is, in effect, an enumeration guide for an attacker.

This duality is important to understand. When you learn what a secure configuration looks like, you simultaneously learn what an insecure one looks like and how to identify it during an engagement.
### What Systems Are Reviewed
Configuration reviews apply to any system that can be audited. Workstations, file servers, domain controllers, web servers, database servers, network appliances, and cloud instances are all subject to configuration review. The specific checks vary by operating system and role, but the categories of misconfiguration are broadly consistent across platforms. A Linux web server and a Windows domain controller both have file permissions, service configurations, user accounts, and credential storage that can be audited against a baseline.

## 3. Security Baselines & Frameworks**

A security baseline is a documented standard that defines how a system should be configured to meet an acceptable level of security. Rather than relying on individual judgement about what constitutes a "secure" setup, organisations adopt baselines that provide specific, measurable recommendations for every configurable aspect of a system, from password policies and file permissions to service configurations and network settings.

Several industry-recognised frameworks publish these baselines. The two most widely referenced are **CIS Benchmarks** and **DISA STIGs**.

### **CIS Benchmarks**

The Center for Internet Security (CIS) publishes configuration benchmarks for a wide range of operating systems, cloud platforms, applications, and network devices. **CIS Benchmarks** are developed through a consensus-driven process involving security professionals from government, industry, and academia. Each benchmark contains hundreds of individual recommendations, each with a description, a rationale explaining the security risk it addresses, an audit procedure for checking compliance, and a remediation procedure for correcting deviations.

**CIS Benchmarks** define two profile levels. Level 1 recommendations are practical, broadly applicable settings that can be implemented on most systems without significant impact on functionality. Level 2 recommendations provide deeper defence-in-depth hardening but may restrict certain functionality or require more careful testing before deployment. An organisation typically selects one of these profiles as its baseline and measures hosts against it.

For example, a CIS Benchmark for Ubuntu Linux might include a Level 1 recommendation that the PermitRootLogin directive in the SSH server configuration must be set to no. The rationale explains that allowing direct root login over SSH provides an attacker who obtains or brute-forces the root password with immediate privileged access, bypassing any accountability provided by individual user accounts. The audit procedure specifies the command to check the current setting in sshd_config, and the remediation procedure provides the configuration change to disable it.

### **DISA STIGs**

The Defense Information Systems Agency (DISA), part of the U.S. Department of Defense, publishes Security Technical Implementation Guides (STIGs). STIGs prescribe hardening requirements for systems used in government and military environments. They are more prescriptive than **CIS Benchmarks** and are mandatory for systems within U.S. government networks.

Each STIG finding is assigned a severity category. CAT I findings represent the highest risk, where exploitation could directly lead to loss of confidentiality, integrity, or availability. CAT II findings represent medium risk, and CAT III findings represent low risk. This categorisation helps administrators prioritise remediation and helps penetration testers prioritise which deviations to investigate first.

### Relationship to Compliance Standards**

**CIS Benchmarks** and **DISA STIGs** do not exist in isolation. Broader compliance frameworks such as PCI-DSS (for payment card processing), ISO 27001 (for information security management), and NIST 800-53 (for federal information systems) frequently reference CIS or STIG hardening as implementation guidance for their own control requirements. When an organisation must demonstrate compliance with PCI-DSS, the specific system hardening it implements is often drawn directly from a CIS Benchmark.

For a penetration tester, awareness of these frameworks serves two purposes. First, they define what the target organisation considers "secure," which means deviations from the adopted baseline are the most defensible findings in a penetration testing report. Second, they provide a structured, comprehensive list of things to check, which is far more reliable than ad-hoc enumeration.

## 4. Automated Compliance Tooling**

Manually auditing a host against a full CIS Benchmark or DISA STIG is time-consuming. A single benchmark can contain several hundred individual recommendations, and enterprise environments may have thousands of hosts to assess. Automated compliance tools address this problem by scanning hosts programmatically against defined baselines and producing reports that flag deviations.

Several tools are widely used for this purpose. Each serves a slightly different audience and use case, but all share the common goal of measuring a host's configuration against a known-good standard.

### **Nessus**

**Nessus** is a commercial vulnerability scanner developed by Tenable. While it is best known for vulnerability scanning, **Nessus** also includes compliance auditing functionality. Administrators can configure **Nessus** to scan hosts against **CIS Benchmarks**, **DISA STIGs**, and custom audit policies. The resulting compliance report lists each check, its pass or fail status, and remediation guidance for failed checks.

From a penetration tester's perspective, a **Nessus** compliance report is a valuable source of information when available. If the target organisation has already run compliance scans internally and shares the results as part of a grey-box or white-box engagement, the report effectively provides a pre-built list of misconfigurations. Each failed check is a potential escalation vector worth investigating.

### **Lynis**

**Lynis** is an open-source security auditing tool designed for Linux, macOS, and other Unix-based systems. Unlike **Nessus**, **Lynis** runs locally on the host being audited rather than scanning remotely. It checks system configuration against hardening best practices and produces a hardening index score along with a categorised list of findings, warnings, and suggestions.

**Lynis** is useful in both defensive and offensive contexts. A system administrator runs **Lynis** to identify hardening gaps on servers under their management. A penetration tester who has gained shell access to a Linux target can run **Lynis** to quickly enumerate configuration weaknesses without manually checking each category. However, uploading or running additional tools on a target during an engagement introduces risk of detection and may violate the engagement's rules of engagement. The decision to use **Lynis** on a target should be weighed against the agreed scope and the operational security requirements of the assessment.

### **OpenSCAP**

**OpenSCAP** is an open-source framework that implements the Security Content Automation Protocol (SCAP), a standardised method for expressing and evaluating security configuration policies. **OpenSCAP** evaluates a host against SCAP-formatted profiles, including CIS and STIG content, and produces detailed reports with pass/fail results for each rule.

**OpenSCAP** is commonly deployed in government and enterprise environments where automated compliance validation is a regulatory requirement. Its output is structured and machine-readable, making it suitable for integration into larger compliance management workflows.

### **CIS-CAT**

**CIS-CAT** is the Center for Internet Security's own assessment tool, purpose-built to evaluate hosts against **CIS Benchmarks**. It is available in both a free limited version (**CIS-CAT** Lite) and a full commercial version (**CIS-CAT** Pro). **CIS-CAT** produces detailed pass/fail reports mapped directly to the benchmark recommendations, making it the most straightforward tool for CIS Benchmark compliance.

### Offensive Enumeration Tools**

The tools described above are designed primarily for defensive compliance auditing. A separate category of tools exists for offensive privilege escalation enumeration. Tools such as **LinPEAS**, **WinPEAS**, and **PowerUp** are designed to run on a compromised host and identify configuration weaknesses from an attacker's perspective. They check for many of the same issues that compliance tools flag, such as writable files, weak service permissions, and stored credentials, but their output is tailored for exploitation rather than remediation.

The overlap between these two categories is significant. A CIS Benchmark check that verifies SSH root login is disabled is functionally related to a **LinPEAS** check that highlights the SSH configuration. The difference lies in the intended audience and the action taken on the finding. The compliance tools and offensive tools covered in later rooms in this module are two sides of the same coin.

## 5. Categories of Misconfiguration

> 🎯 **Enumeration Checklist:** Work through the six categories systematically. Do not rely only on intuition or automated tools.
**

Host misconfigurations that lead to privilege escalation fall into a set of well-defined categories. Understanding these categories provides a mental framework for approaching any system, regardless of its operating system or role. Each category represents a type of deviation from secure baseline configuration that an attacker can exploit.

This task describes each category at a conceptual level. The specific techniques for identifying and exploiting these misconfigurations on Linux and Windows are covered in the dedicated rooms later in this module.

### User and Group Configuration**

Operating systems use accounts and groups to control who can perform which actions. A secure baseline defines the principle of least privilege, where each account has only the permissions necessary for its intended function. Deviations from this principle create escalation opportunities.

Common misconfigurations in this category include assigning user accounts to administrative groups unnecessarily, creating service accounts with excessive privileges, failing to disable or remove default accounts that ship with the operating system, and configuring weak or absent password policies that allow easily guessable credentials. An attacker who compromises a user account that belongs to an administrative group, for instance, may already have elevated privileges without needing any further exploitation.

### File and Directory Permissions**

File and directory permissions control which users can read, write, or execute specific files. Both Linux and Windows implement access control mechanisms for this purpose, though the models differ in their specifics. A secure baseline defines strict permissions on sensitive files, system binaries, and configuration directories.

On Linux, misconfigurations in this category include the **SUID** (Set User ID) bit being set on binaries that do not require it. The **SUID** bit is a special permission that causes an executable to run with the privileges of the file's owner rather than the user who executes it. When a **SUID** binary is owned by root and contains functionality that allows arbitrary command execution or file access, it becomes a direct privilege escalation vector. Other common issues include world-writable scripts or binaries that are executed by privileged processes, and sensitive files such as /etc/shadow or SSH private keys having read permissions that are too broad.

On Windows, the analogous issues involve Access Control Lists (**ACLs**) that grant excessive permissions. Directories in the system PATH that are writable by non-privileged users, program installation directories with weak default permissions, and misconfigured registry key **ACLs** all fall into this category.

### Service Configurations**

Services are long-running processes that perform background tasks on a system. On Linux, services are typically managed by systemd and defined in unit files. On Windows, services are managed by the Service Control Manager (SCM) and configured through the registry and service properties.

A secure baseline requires services to run with the minimum privileges necessary, to be configured with restrictive access controls preventing unauthorised modification, and to reference their executables using full, unambiguous paths.

Common misconfigurations include services that run as root or LocalSystem when they do not need to, service configuration files or binaries that are writable by non-privileged users, and service binary paths that are unquoted and contain spaces. On Windows, the unquoted service path issue is particularly well-known. When a service's executable path contains spaces and is not enclosed in quotation marks, the Service Control Manager attempts to resolve the path by testing each possible token boundary. For a service with the path C:\Program Files\My App\service.exe, Windows will first attempt to execute C:\Program.exe, then C:\Program Files\My.exe, and then C:\Program Files\My App\service.exe. If a non-privileged user can write to any of the intermediate directories, they can place a malicious binary at one of the tested paths, and it will execute with the privileges of the service account.

### Scheduled Tasks and Cron Jobs**

Both Linux and Windows support scheduling commands or scripts to run automatically at defined intervals. On Linux, this is handled by cron, a job scheduler that executes commands according to time-based rules defined in crontab files. On Windows, the Task Scheduler provides equivalent functionality.

Scheduled tasks that run with elevated privileges and reference scripts or binaries that a non-privileged user can modify are a direct escalation vector. The secure baseline requires that any script or binary executed by a privileged scheduled task is itself protected by appropriate permissions, and that the task's configuration is not modifiable by unauthorised users.

A secondary issue involves the behaviour of certain utilities when processing filenames that resemble command-line flags. A scheduled task that uses a wildcard to process files in a directory may be vulnerable to wildcard injection. For example, if a root cron job runs tar cf /backup/archive.tar * inside a directory, an attacker who can create files in that directory can add files named --checkpoint=1 and --checkpoint-action=exec=sh shell.sh. When tar processes the wildcard, it interprets these filenames as command-line arguments and executes the attacker's script with the privileges of the cron job.

### Credential Storage**

Administrators and users frequently store credentials in locations that are accessible to other accounts on the system. This is not a permission misconfiguration in the traditional sense but rather an operational practice that violates the principle of credential hygiene.

On Linux, common locations for exposed credentials include shell history files such as ~/.bash_history, environment variables, application configuration files containing database connection strings or API keys, and SSH private keys stored with overly permissive read access.

On Windows, credential exposure may involve entries in the Windows Credential Manager (accessible via 

```bash

```bash
cmdkey /list
```

```

), saved credentials used with runas /savecred, cleartext passwords in the Windows registry, deployment files such as Unattend.xml and Sysprep configuration files, PowerShell command history, and web application configuration files such as web.config.

An attacker who discovers stored credentials for a privileged account can escalate directly without exploiting any technical misconfiguration in the system itself.

### Network Configuration**

Network configuration misconfigurations are less commonly associated with local privilege escalation than the categories above, but they remain relevant to the overall security posture of a host. This category is included for completeness and is not a primary focus of this module.

A secure baseline defines which ports and services should be accessible, from which interfaces, and to which source addresses. Deviations from this baseline expand the attack surface available to an attacker who has already gained a foothold on the network. For example, a database service such as MySQL bound to 0.0.0.0 (all interfaces) instead of 127.0.0.1 (localhost only) is accessible from the network rather than restricted to local connections. If the database has weak credentials or no authentication, this misconfiguration provides a path to data access or, in some cases, command execution through database-specific features such as MySQL's INTO OUTFILE or PostgreSQL's COPY TO PROGRAM.

Other common issues include unnecessary ports left open, overly permissive firewall rules that allow inbound connections to management interfaces, and insecure remote management protocols that transmit credentials in cleartext.

## 6. Structured Enumeration Methodology

> 🧠 **Mental Model:** `Situational Awareness → Category Enumeration → Prioritisation → Chaining → Validation`
**

Approaching a host without a defined methodology leads to inconsistent results. Testers who rely on memory or intuition will inevitably overlook categories of misconfiguration, particularly under time pressure. A structured enumeration methodology ensures comprehensive coverage by mapping each step to a defined category of misconfiguration.

The methodology presented here is operating-system-agnostic. It defines what to check rather than how to check it. The specific commands and tools for Linux and Windows are covered in the dedicated rooms that follow in this module.
### Phase 1 — Situational Awareness
Before checking for specific misconfigurations, you need to understand the context of the host. This initial phase answers fundamental questions about the system and the account you are operating under.

You should determine the identity and privileges of the current user account, including group memberships. You should identify the operating system, its version, and its architecture. You should determine the hostname and understand the host's role within the network, whether it is a workstation, a web server, a database server, or a domain controller. You should also note whether the host is domain-joined, as this affects which escalation paths are available.

This information shapes the enumeration that follows. A domain-joined Windows server running IIS presents different opportunities than a standalone Linux workstation. Knowing what you are working with prevents wasted effort.
### Phase 2 — Category-Based Enumeration
With situational awareness established, enumeration proceeds through each misconfiguration category identified in the previous task. The order is not strictly fixed, but a consistent sequence ensures nothing is missed.

User and group configuration. Enumerate all user accounts on the system and their group memberships. Identify which accounts have administrative or root-level access. Check for default accounts that should have been disabled. Review password policy settings where accessible.

File and directory permissions. Search for files and directories with overly permissive access controls. On Linux, this includes locating **SUID** and **SGID** binaries, world-writable files and directories, and sensitive files with broad read permissions. On Windows, this includes reviewing **ACLs** on directories in the system PATH, program installation directories, and registry keys associated with services.

Service configurations. List all running services and their configurations. Identify which user account each service runs as. Check whether the service's binary or configuration file is writable by the current user. On Windows, check for unquoted service paths and review service security descriptors.

Scheduled tasks and cron jobs. Enumerate all scheduled jobs and identify those running with elevated privileges. Check the permissions on any scripts or binaries referenced by privileged scheduled tasks. On Linux, review system-wide and user-specific crontab entries. On Windows, review tasks in the Task Scheduler.

Credential storage. Search for credentials stored in accessible locations. Check shell history files, environment variables, configuration files, registry entries, credential stores, and deployment files. This step often yields the most direct path to escalation.

Network configuration. Review listening ports, bound interfaces, and firewall rules. Identify services that are unnecessarily exposed or accessible. This step is more relevant to lateral movement than local escalation, but it contributes to the overall understanding of the host's posture.
### Phase 3 — Prioritisation & Exploitation
Enumeration typically produces multiple findings. Not all findings are equally exploitable or equally impactful. Prioritise findings based on the directness of the escalation path, with a preference for findings that provide the most reliable route to the highest privilege level.

Stored credentials for a root or administrator account, for instance, represent a more direct escalation path than a writable file in a cron job that executes hourly. Both are valid findings, but the first requires no waiting or additional steps.

Some findings are not exploitable in isolation but become significant when combined. For example, enumeration might reveal that a root cron job runs /opt/scripts/backup.sh every five minutes (a scheduled task finding) and that the script is world-writable (a file permissions finding). Neither finding is sufficient on its own. The cron job is not inherently insecure, and a world-writable file that is never executed by a privileged process is a low-priority issue. However, the combination of the two provides a direct privilege escalation path: modify the script, wait for the cron job to execute it as root, and gain elevated access. Recognising these chains across categories is a core skill in privilege escalation methodology.
### The Role of Automated Tools
Automated enumeration tools such as **LinPEAS**, **WinPEAS**, and **PowerUp** map directly to this methodology. They perform many of the checks described above and highlight findings by severity. However, they are a supplement to the methodology, not a replacement. Understanding what the tools are checking and why allows you to interpret their output correctly, identify false positives, and recognise gaps where manual investigation is required.

If compliance scan output from tools such as **Nessus** or **Lynis** is available, whether through a white-box engagement or through running **Lynis** on a compromised host, its findings can be used to prioritise which categories to investigate first. A failed CIS Benchmark check on service permissions, for instance, points directly to the service configurations category.

--Ad-hoc scanning leads to missed findings. A structured Local Privilege Escalation (LPE) methodology ensures you systematically evaluate every vector on a target host, whether using manual commands or automated scripts.

Here is how the 3-phase enumeration process works in practice:
### The 3-Phase Enumeration Workflow
┌────────────────────────────────────────────────────────────────────────┐

│                   PHASE 1: SITUATIONAL AWARENESS                      │

│ Identify: User identity, Groups, OS version, Kernel, Domain/Workgroup   │

└──────────────────────────────────┬─────────────────────────────────────┘

                                   │

                                   ▼

┌────────────────────────────────────────────────────────────────────────┐

│                 PHASE 2: CATEGORY-BASED ENUMERATION                    │

│ 1. User & Group Perms  ──► Sudo rights, Admin groups                   │

│ 2. File & Directory    ──► **SUID**/**SGID** bits, Weak **ACLs**                   │

│ 3. Service Configs     ──► Unquoted paths, writable service binaries   │

│ 4. Scheduled Tasks     ──► Crontabs, Task Scheduler                    │

│ 5. Credential Storage  ──► Shell history, config files, SAM/Vault      │

│ 6. Network Config      ──► Internal-only listening ports (127.0.0.1)    │

└──────────────────────────────────┬─────────────────────────────────────┘

                                   │

                                   ▼

┌────────────────────────────────────────────────────────────────────────┐

│               PHASE 3: PRIORITISATION & CHAINING                       │

│ High-Priority: Cleartext root passwords in history/config files       │

│ Medium-Priority: Vulnerable **SUID** binary, writable service path         │

│ Low-Priority: Hourly cron job (needs chaining with weak file perms)    │

└────────────────────────────────────────────────────────────────────────┘

Phase 1: Situational Awareness

Before running exploitation checks, answer these core questions:

Who am I? (

```bash

```bash
whoami
```

```

, 

```bash

```bash
id
```

```

, 

```bash

```bash
whoami
```

```

 /priv, net user \<username>)

What am I running on? (

```bash

```bash
uname -a
```

```

, 

```bash
cat /etc/os-release
```

, 

```bash

```bash
systeminfo
```

```

)

Is this machine domain-joined? (Determines if Active Directory attacks apply or purely local LPE).

Phase 2: Category-Based Enumeration

Work through the 5 main categories methodically. Whether running manual checks or interpreting automated script output (like **LinPEAS**/**WinPEAS**), organize your notes by these exact buckets:

User & Group Configuration: Are you part of special groups (e.g., docker, disk, lxd, Backup Operators) that grant indirect root access?

File & Directory Permissions: Search for **SUID** binaries on Linux or weak folder **ACLs** on Windows (

```bash

```bash
icacls
```

```

).

Service Configurations: Look for service binaries running as root/SYSTEM where you have write permissions to the executable or directory.

Scheduled Tasks & Cron Jobs: List system-wide crontabs (

```bash

```bash
cat /etc/crontab
```

```

, /etc/cron.*) and Windows Task Scheduler entries.

Credential Storage: Check shell history (~/.bash_history, PowerShell ConsoleHost_history.txt), environment variables (

```bash

```bash
env
```

```

), and configuration files (/var/www/html/).

Phase 3: Prioritisation and Exploitation Chaining

Not all findings are equal. Sort your findings by speed and reliability:

Direct Escalation (Highest Priority): Hardcoded root password found in a .

```bash

```bash
env
```

```

 or history file. Requires zero risk and instant execution.

Exploitation Chaining (Medium/High Priority): Individual findings that mean little alone, but create a complete attack path when linked together.

Example of Exploitation Chaining:

  [Finding A: Category 4]                [Finding B: Category 2]

Privileged Cron job executes           The script file \`/opt/backup.sh\`

 \`/opt/backup.sh\` every 2 mins          has \`world-writable\` (777) permissions

            │                                      │

            └──────────────────┬───────────────────┘

                               ▼

                       CHAINED ATTACK

          Append reverse shell command to \`/opt/backup.sh\`

                               │

                               ▼

            ROOT SHELL OBTAINED IN < 2 MINUTES

## 7. Reading a CIS Benchmark

> 🔎 **Offensive Mindset:** Read a security recommendation as both a defender and a penetration tester: **What should be true? → Is it true? → If not, what can the deviation enable?**
**

Understanding the structure of a CIS Benchmark recommendation is a practical skill for both offensive and defensive work. Each recommendation follows a consistent format that, once familiar, allows you to quickly assess what a check is protecting against and what an exploitable deviation looks like.
### Structure of a Recommendation
A typical CIS Benchmark recommendation contains several standard fields.

The title identifies the specific setting or configuration being assessed. It is usually phrased as an imperative or declarative statement. An example from the CIS Benchmark for Ubuntu Linux is "Ensure permissions on /etc/shadow are configured."

The profile applicability indicates whether the recommendation belongs to the Level 1 or Level 2 profile. As described in Task 3, Level 1 recommendations are broadly applicable with minimal risk of disrupting functionality, while Level 2 recommendations provide additional hardening at the cost of potential operational impact.

The description explains what the recommendation requires in plain terms. For the /etc/shadow example, the description states that the file should be owned by root, with its group set to shadow, and its permissions set so that only root has read and write access while the shadow group has read access.

The rationale explains the security risk that the recommendation addresses. In this case, the rationale explains that /etc/shadow contains hashed passwords for all local user accounts, and that if non-privileged users can read this file, they can extract the hashes and attempt offline cracking to recover plaintext passwords.

The audit section provides specific commands or procedures to verify whether the host complies with the recommendation. For the /etc/shadow example, the audit procedure specifies running 

```bash

```bash
stat /etc/shadow
```

```

 and verifying that the output shows the expected ownership and permission values.

The remediation section provides the commands or steps to bring a non-compliant host into compliance. For this example, it would specify 

```bash
chown and chmod
```

 commands to correct the ownership and permissions.
### Reading from an Offensive Perspective
A penetration tester reads the same recommendation with a different question in mind. Rather than asking "is this system compliant?", the question is "if this system is not compliant, what can I do with the finding?"

For the /etc/shadow example, the offensive reading is straightforward. If the audit check fails and /etc/shadow is readable by the current user, you can copy the file, extract the password hashes, and run an offline cracking attack using a tool such as John the Ripper or Hashcat. Successful cracking of a root or administrative user's hash provides direct privilege escalation.

Not every failed benchmark check leads to a direct escalation path. Some findings represent defence-in-depth improvements that, in isolation, are not exploitable. The skill lies in recognising which deviations are directly actionable, which contribute to a chain of exploitation when combined with other findings, and which are low-priority informational issues.
### Mapping to Automated Tool Output
The checks performed by automated compliance tools correspond directly to individual benchmark recommendations. A compliance scan that flags "7.1.5 - Ensure permissions on /etc/shadow are configured - FAILED" is reporting the result of the same audit procedure described in the CIS Ubuntu Linux 24.04 LTS Benchmark. Similarly, a **LinPEAS** scan that highlights /etc/shadow as readable by the current user is detecting the same issue from an offensive perspective.

Understanding the benchmark recommendation behind a tool finding allows you to assess its severity and exploitability with greater confidence than relying on the tool's output alone.

--A CIS Benchmark recommendation is structured to give system administrators everything they need to audit and fix a setting. However, as an ethical hacker or penetration tester, you read that exact same document to find what to break and exploit.Understanding this structure helps you bridge the gap between compliance reports (like **Nessus**/**OpenSCAP**) and offensive exploitation.Anatomy of a CIS Recommendation: Dual PerspectivesHere is how a single CIS check (e.g., /etc/shadow file permissions) is structured, and how the Blue Team (Defense) and Red Team (Offense) interpret each section:                  CIS BENCHMARK RECOMMENDATION

                               │

   ┌───────────────────────────┴───────────────────────────┐

   ▼                                                       ▼

DEFENSIVE PERSPECTIVE (Blue)                    OFFENSIVE PERSPECTIVE (Red)

• Title: What setting to check?                 • Finding: What standard is missing?

• Description: How should it look?              • Baseline Deviation: Is this misconfigured?

• Rationale: Why is this dangerous?             • Impact: How can I turn this into access?

• Audit: Command to check compliance.           • Discovery: Manual command to verify flaw.

• Remediation: Command to fix the issue.        • Action: Exact attack vector to execute.

Breakdown of the Standard CIS FieldsCIS SectionDefensive Intent (SysAdmin / Auditor)Offensive Intent (Penetration Tester)TitleIdentifies the rule (e.g., 7.1.5 Ensure permissions on /etc/shadow are configured).Tells you what component to target.ProfileDetermines if it is Level 1 (Basic/Safe) or Level 2 (High Security/Strict).Level 1 failures mean basic security was missed; Level 2 failures mean defense-in-depth is lacking.DescriptionStates expected state (e.g., root:shadow ownership, 0640 permissions).Tells you what "bad" looks like (e.g., 0644 or world-readable).RationaleExplains the security risk (e.g., prevents local users from reading password hashes).Your Attack Scenario: Tells you why this failure is useful (e.g., extract hashes $\rightarrow$ offline crack with Hashcat).AuditTerminal command to test compliance (e.g., 

```bash

```bash
stat /etc/shadow
```

```

).The exact command to run during manual enumeration to confirm the vulnerability.RemediationCommands to lock down the file (e.g., 

```bash
chmod 640 /etc/shadow
```

).Not used directly, but shows what steps the client must take in your final report.Triaging Compliance Findings: Direct vs. Chained AttacksWhen reviewing scan output (from **Nessus**, **OpenSCAP**, or **LinPEAS**), classify failed checks into three operational tiers:┌─────────────────────────────────────────────────────────────────────────┐

│ 1. DIRECT ESCALATION (High Priority)                                   │

│ Example: /etc/shadow is world-readable or /etc/passwd is world-writable.│

│ Result: Immediate root shell or instant hash extraction.                │

└────────────────────────────────────┬────────────────────────────────────┘

                                     │

                                     ▼

┌─────────────────────────────────────────────────────────────────────────┐

│ 2. CHAINED ESCALATION (Medium Priority)                                 │

│ Example: Sudo version is outdated + non-standard binary has **SUID** bit.   │

│ Result: Requires combining two distinct findings to build an exploit.   │

└────────────────────────────────────┬────────────────────────────────────┘

                                     │

                                     ▼

┌─────────────────────────────────────────────────────────────────────────┐

│ 3. DEFENSE-IN-DEPTH / HARDENING (Low Priority)                          │

│ Example: Single-user mode password missing, or core dumps enabled.      │

│ Result: Good practice to fix, but rarely provides an actionable shell.  │

└─────────────────────────────────────────────────────────────────────────┘

Example: Mapping a Failed Rule to an AttackIf an **OpenSCAP** or **Nessus** report flags:FAIL: Rule 5.2.14 - Ensure SSH PermitRootLogin is set to noDefensive View: Edit /etc/ssh/sshd_config, change PermitRootLogin yes to no, and restart sshd.Offensive View: If you brute-force or discover the root account password during an internal assessment, you do not need to hop through a low-privilege account first—you can SSH directly as root.

8\.

---

## 🧠 Final Takeaways

> **A secure baseline is an attacker's checklist.**

When you land on a host, ask:

1. **Who am I?**
2. **What system am I on?**
3. **What privileges and groups do I have?**
4. **What users and groups exist?**
5. **What files or ACLs are weak?**
6. **What privileged services are running?**
7. **What scheduled tasks or cron jobs execute with elevated privileges?**
8. **Where are credentials stored?**
9. **What network services are exposed?**
10. **Can two or more findings be chained?**

### 🔁 Revision Formula

```text
ENUMERATE
    ↓
IDENTIFY MISCONFIGURATION
    ↓
ASSESS PRIVILEGE CONTEXT
    ↓
CHECK EXPLOITABILITY
    ↓
CHAIN FINDINGS IF NEEDED
    ↓
VALIDATE ACCESS
    ↓
DOCUMENT THE ATTACK PATH
```

> **Next step:** Use the Linux and Windows Privilege Escalation notes for the OS-specific commands and practical exploitation techniques.
