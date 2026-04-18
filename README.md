# 🔬 Home Security Lab Setup

> A personal virtualized security lab built for hands-on practice in threat detection,
> network traffic analysis, and offensive security fundamentals — in a fully isolated environment.

---

## 📋 Lab Overview

This lab simulates a small enterprise-like environment where I can:
- Launch attacks from a dedicated attacker machine
- Capture and analyze network traffic
- Ingest logs into a SIEM and practice writing detections
- Practice with real security tools in a safe, isolated space

All activity documented here is performed **exclusively within this isolated lab**.
No tools or techniques have been used against any unauthorized systems.

---

## 🖥️ Lab Architecture

| Machine | OS | Role | RAM Allocated |
|---|---|---|---|
| Attacker | Kali Linux (latest) | Penetration testing, recon, attack simulation | 4GB |
| Target | Metasploitable 2 | Intentionally vulnerable target machine | 1GB |
| Host | Windows (physical) | Runs VirtualBox, accesses Splunk dashboard via browser | 16GB total |

**Virtualization Platform:** Oracle VirtualBox

**Host Machine RAM:** 16GB

---

## 🌐 Network Topology

```
┌─────────────────────────────────────────────┐
│              HOST MACHINE (Windows)         │
│                   16GB RAM                  │
│                                             │
│  ┌──────────────────┐  ┌─────────────────┐  │
│  │   Kali Linux VM  │  │ Metasploitable  │  │
│  │   (Attacker +    │  │      2 VM       │  │
│  │     Splunk)      │  │   (Target)      │  │
│  │                  │  │                 │  │
│  │  Adapter 1: NAT  │  │  Adapter 1:     │  │
│  │  (internet       │  │  Host-Only      │  │
│  │   access)        │  │  (isolated)     │  │
│  │                  │  │                 │  │
│  │  Adapter 2:      │  │                 │  │
│  │  Host-Only       │  │                 │  │
│  │  (lab network)   │  │                 │  │
│  └──────────────────┘  └─────────────────┘  │
│                                             │
│         Host-Only Network (isolated)        │
│         192.168.56.0/24                     │
└─────────────────────────────────────────────┘
```

**Network Design Reasoning:**
- Kali has **two adapters**: NAT (for downloading tools and updates) and Host-Only (for attacking Metasploitable)
- Metasploitable has **only Host-Only**: deliberately no internet access — it is intentionally vulnerable and must never reach the internet
- This design mirrors real SOC environments where monitored endpoints are segmented from general internet access

> 📁 See [`network-diagram/`](./network-diagram/) for the visual diagram

---

## 🛠️ Tools Installed

### On Kali Linux
| Tool | Category | Purpose |
|---|---|---|
| Splunk Free | SIEM | Log ingestion, search, detection, dashboards |
| Wireshark | Network Analysis | Packet capture and traffic analysis |
| Nmap / Zenmap | Recon | Network scanning and service enumeration |
| Metasploit Framework | Exploitation (lab) | Attack simulation against Metasploitable |
| SQLmap | Web Testing (lab) | SQL injection testing |
| Tcpdump | Network Analysis | CLI-based packet capture |
| Burp Suite Community | Web Proxy (lab) | HTTP traffic interception |

> 📁 See [`security-tools`](https://github.com/Haadi-Balouch/security-tools) for setup notes on each tool (under progress)

---

## 📁 Repository Structure

```
home-lab-setup/
├── README.md                        ← You are here
├── network-diagram/
│   └── lab-topology.png             ← Visual network diagram
├── vm-setup/
│   ├── kali-linux-setup.md          ← Kali VM configuration notes
│   └── metasploitable-setup.md      ← Metasploitable 2 setup and service inventory
├── screenshots/
│   └── png pictures
└── lessons-learned.md               ← Honest notes on problems hit and solved
```

---

## 🔗 Related Repositories

This lab is the foundation for all detection and analysis work in my portfolio:

| Repo | How it uses this lab |
|---|---|
| [siem-detection-lab](../siem-detection-lab) | All Splunk detections run against logs generated here |
| [security-tools](../security-tools) | All tool notes are based on hands-on use in this environment |

---

## 🗺️ Planned Additions

- [ ] Add Windows 10 VM as a monitored endpoint (Windows Event Log forwarding to Splunk)
- [ ] Install Wazuh alongside Splunk for XDR/EDR practice
- [ ] Add pfSense VM as a network firewall between attacker and target
- [ ] Set up a vulnerable web app (DVWA) for web application testing

---

## 📚 Resources Used to Build This Lab

- [VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)
- [Splunk Free Download](https://www.splunk.com/en_us/download/splunk-enterprise.html)
- [Metasploitable 2 Setup Guide — Rapid7](https://docs.rapid7.com/metasploit/metasploitable-2/)
