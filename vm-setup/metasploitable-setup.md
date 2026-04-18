# Metasploitable 2 — Setup Notes

## What Is Metasploitable 2?

Metasploitable 2 is an intentionally vulnerable Linux virtual machine created by Rapid7
(the company behind Metasploit). It is designed specifically to be attacked — it runs
outdated software with known vulnerabilities across multiple services.

It is the standard practice target machine for learning penetration testing and,
from a blue team perspective, for generating realistic attack traffic to detect in a SIEM.

---

## VM Configuration

| Setting | Value |
|---|---|
| OS | Ubuntu Linux 8.04 (intentionally outdated) |
| Virtualization | Oracle VirtualBox |
| RAM Allocated | 1GB |
| Storage | 8GB |
| Network Adapter | Host-Only (vboxnet0) ONLY |
| Default Credentials | `msfadmin` / `msfadmin` |

---

## ⚠️ Critical Security Note

**Metasploitable 2 is deliberately vulnerable. It must NEVER be connected to the internet or a bridged/NAT network.**

This is why it is configured with Host-Only networking only. It can only communicate with Kali Linux on the isolated `192.168.56.0/24` network. The host machine and the internet are completely unreachable from Metasploitable.

Before starting Metasploitable, always verify its network adapter is set to Host-Only in VirtualBox settings.

---

## Network Configuration

**IP Address:** `192.168.56.102` (assigned by VirtualBox DHCP on Host-Only network)

To verify from inside Metasploitable (login with msfadmin/msfadmin):
```bash
ifconfig
```

To verify connectivity from Kali:
```bash
ping 192.168.56.102
nmap -sn 192.168.56.0/24     # scan the whole host-only subnet
```

---

## Service Inventory (Nmap Scan Results)

Running a full service scan from Kali against Metasploitable reveals the attack surface:

```bash
nmap -sV -O 192.168.56.102
```

> [Add your actual nmap scan output here as a screenshot or paste the text results]
> Example of what you will see:

| Port | Service | Version | Known Vulnerability |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 | Backdoor command execution (CVE-2011-2523) |
| 22/tcp | SSH | OpenSSH 4.7p1 | Weak algorithms, brute-forceable |
| 23/tcp | Telnet | Linux telnetd | Cleartext credentials |
| 25/tcp | SMTP | Postfix smtpd | Open relay possible |
| 80/tcp | HTTP | Apache 2.2.8 | Multiple web app vulns (DVWA, phpMyAdmin) |
| 139/tcp | NetBIOS | Samba 3.x | Username enumeration, MS08-067 style issues |
| 445/tcp | SMB | Samba 3.0.20 | Username map script RCE (CVE-2007-2447) |
| 3306/tcp | MySQL | MySQL 5.0.51a | Default root with no password |
| 5432/tcp | PostgreSQL | PostgreSQL 8.3 | Default credentials |
| 8180/tcp | HTTP | Apache Tomcat 5.5 | Default manager credentials |

---

## Why Each Service Matters for SOC Training

Each vulnerable service on Metasploitable teaches a different detection skill:

- **vsftpd backdoor (port 21):** Teaches detecting unusual outbound connections spawned by a service
- **SSH (port 22):** Teaches brute force detection — multiple failed auth events in SIEM
- **SMB (port 445):** Teaches lateral movement detection — SMB exploitation is extremely common in real attacks
- **MySQL (port 3306):** Teaches database attack detection — unauthorized DB access patterns
- **HTTP/DVWA (port 80):** Teaches web attack detection — SQLi, XSS, command injection in HTTP logs

---

## Common Issues Hit During Setup

**Issue 1 — Metasploitable wouldn't boot, showed kernel panic**
- Root cause: VirtualBox version incompatibility with the old Metasploitable disk image
- Fix: In VirtualBox VM settings → System → set to "Other Linux (32-bit)" and disabled EFI

**Issue 2 — Could not ping Metasploitable from Kali**
- Root cause: Host-Only network DHCP wasn't assigning an IP to Metasploitable
- Fix: Verified VirtualBox Host Network Manager had DHCP enabled for vboxnet0. Rebooted Metasploitable VM.

---

## References

- [Metasploitable 2 Official Guide — Rapid7](https://docs.rapid7.com/metasploit/metasploitable-2/)
- [Metasploitable 2 Exploitability Guide](https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/)
