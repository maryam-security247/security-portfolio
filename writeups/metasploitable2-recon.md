# Metasploitable2 - Backdoor Exploitation via Port 1524

## Date
September 2026

## Objective
Perform reconnaissance and exploit any discovered vulnerabilities against 
a Metasploitable2 VM in an isolated local lab environment.

## Environment
- Attacker machine: Kali Linux (192.168.10.128)
- Target: Metasploitable2 VM (192.168.10.129)
- Network: 192.168.10.0/24 (isolated VMware host-only network)

## Phase 1: Reconnaissance

### Host Discovery
```bash
nmap -sn 192.168.10.0/24
```
Identified 4 active hosts, including the target at 192.168.10.129.

### Service/Version Scan
```bash
nmap -sV 192.168.10.129
```
Confirmed the target as `metasploitable.localdomain` with 23 open ports, 
including several critically outdated services:

| Port | Service | Risk Notes |
|------|---------|-----------|
| 21   | vsftpd 2.3.4 | Known backdoor (CVE-2011-2523) |
| 22   | OpenSSH 4.7p1 | Outdated |
| 1524 | Unidentified backdoor service | Flagged for further investigation |
| 3306 | MySQL 5.0.51a | Outdated |
| 6667 | UnrealIRCd | Historically vulnerable |

## Phase 2: Exploitation

Port 1524 appeared unusual in the scan (labeled "Metasploitable root shell" 
by Nmap's service detection). I connected directly using Netcat to investigate:

```bash
nc 192.168.10.129 1524
```

**Result:** The connection immediately returned an interactive root shell 
— with **no authentication required whatsoever**.


## Phase 3: Impact Assessment

With unauthenticated root access confirmed, I demonstrated the potential 
impact by retrieving sensitive system files:

**User enumeration** (`/etc/passwd`):
Revealed 30+ system and application accounts, including `msfadmin`, 
`postgres`, `mysql`, and multiple standard user accounts.

**Password hash extraction** (`/etc/shadow`):
Successfully retrieved password hashes for root and several other users. 
Hashes are stored using the outdated and weak **MD5** algorithm (identified 
by the `$1$` prefix), making them highly susceptible to offline cracking 
with tools like John the Ripper or Hashcat.

**System fingerprinting** (`uname -a`):

Confirmed a Linux kernel from 2008 — over 18 years out of date, with 
numerous long-since-patched vulnerabilities.

## Risk Rating
**Critical** — Unauthenticated remote root access allows complete 
system compromise, including full read access to password hashes, 
system configuration, and the ability to execute arbitrary commands 
as the highest-privileged user on the system.

## Recommended Remediation
1. Immediately disable/remove the backdoor service on port 1524
2. Restrict network access to management ports via firewall rules
3. Upgrade to a supported, patched operating system
4. Migrate password hashing to a modern algorithm (bcrypt/SHA-512)
5. Conduct a full audit of all exposed services identified in the scan

## Lessons Learned
This exercise demonstrated how a single unauthenticated backdoor service 
can lead to complete system compromise, bypassing all other security 
controls entirely. It reinforced the importance of thorough port 
enumeration — even seemingly obscure ports can represent critical risks.
