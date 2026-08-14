# 🔑 Credential Discovery

> **Topic:** Introduction to AD Breaching
>
> **Section:** Credential Discovery

---

# 📖 Overview

One of the most effective ways to obtain initial credentials is to search for credentials that developers and administrators have accidentally exposed.

Internal services such as:

- Git repositories
- CI/CD platforms
- File shares

can contain credentials in plaintext.

This maps to:

```text
MITRE ATT&CK T1552
Unsecured Credentials
```

---

# 💎 Why Exposed Services Are a Goldmine

Developers and administrators regularly work with:

- Database connection strings
- Service-account passwords
- API keys
- Deployment secrets

These can accidentally end up:

- In source-code repositories
- In CI/CD build logs
- In configuration files
- In internal documentation

Even after cleanup, traces may remain.

For example:

> A password removed from the latest configuration file can still exist in Git commit history.

---

# 🐙 Hunting Credentials in Git Repositories

Git repositories can expose credentials through several locations.

## Commit History

Credentials may be committed temporarily and removed later.

The secret remains in the Git history.

---

## Configuration Files

Common files to inspect include:

```text
.env
web.config
appsettings.json
config.php
database.yml
```

These may contain:

- Database passwords
- API keys
- Connection strings

---

## Hardcoded Secrets

Credentials may be embedded directly in source code.

This is especially common in:

- Development code
- Testing code
- Temporary scripts

---

## CI/CD Definitions

Potential files include:

```text
Jenkinsfile
.gitlab-ci.yml
.github/workflows/*.yml
```

These can reveal credentials or how secrets are stored.

---

# 🔍 Manual Git History Search

Use:

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

The:

```text
-p
```

option displays commit diffs, allowing previous versions of files to be searched.

---

# 🐷 TruffleHog

TruffleHog can scan an entire repository's commit history for:

- High-entropy strings
- Known credential patterns
- Potential secrets

Example:

```bash
trufflehog git file:///path/to/repo
```

---

# 🔧 Hunting Credentials in Jenkins

Jenkins is a commonly encountered CI/CD platform on internal networks.

Potential credential locations include:

- Build console output
- Job configuration
- Environment variables
- Workspace files

---

## Build Console Output

Build logs may contain:

- Environment variables
- Connection strings
- Deployment commands
- Credentials

Even when Jenkins masks credentials, improper scripting can sometimes expose them.

---

## Job Configuration

A job's:

```text
config.xml
```

may contain hardcoded credentials, especially in older or legacy jobs.

---

## Environment Variables

Jenkins exposes environment variables to build steps.

Commands such as:

```text
env
```

or:

```text
set
```

may expose stored secrets if printed.

---

## Workspace Files

Build workspaces may contain:

- Source code
- Configuration files
- Deployment artefacts

These can contain credentials.

---

# 🔍 Jenkins Build Log Search

The supplied lab material uses:

```bash
curl http://ci.thm.loc/job/JOB_NAME/lastBuild/consoleText | grep -i "password\|secret\|token\|credential"
```

---

# 🧪 Practical Exercise

The lab provides two exposed services.

## Git Repository

```text
http://git.thm.loc/megacorp-admin/webapp-deploy
```

Inspect:

- Repository contents
- Commit history
- Previous changes

---

## Jenkins

```text
http://ci.thm.loc/
```

Lab credentials:

```text
admin:admin
```

Inspect:

- Build history
- Console output
- Job configuration

---

# 🔎 Other Credential Sources

The source also identifies other places worth checking:

## Internal Wikis

Onboarding guides and runbooks may contain:

- Default passwords
- Service-account credentials

## Configuration Files on Network Shares

Potential files include:

```text
web.config
bootstrap.ini
unattend.xml
```

## LDAP Anonymous Binds

Some Domain Controllers may allow unauthenticated LDAP queries.

These can reveal:

- User accounts
- Attributes

## SNMP Community Strings

Default SNMP community strings can expose device configurations containing credentials.

---

# 💡 Key Takeaways

- Exposed services can be an easy path to initial credentials.
- Git history can preserve secrets even after they are removed from current files.
- Configuration files frequently contain sensitive information.
- CI/CD systems can expose secrets through logs and configuration.
- TruffleHog automates credential discovery in Git repositories.
- Jenkins should be checked for leaked credentials in builds, jobs, environment variables and workspaces.
- Partial findings are still valuable because usernames can be added to later password-spraying lists.
- Unsecured credentials map to MITRE ATT&CK T1552.
