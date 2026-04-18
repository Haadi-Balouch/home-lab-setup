# Kali Linux VM — Setup Notes

## VM Configuration

| Setting | Value |
|---|---|
| OS | Kali Linux (latest rolling release) |
| Virtualization | Oracle VirtualBox |
| RAM Allocated | 4GB |
| Storage | 80GB (dynamically allocated) |
| Network Adapter 1 | NAT — internet access for updates and tool downloads |
| Network Adapter 2 | Host-Only (vboxnet0) — isolated lab network for attacking Metasploitable |
| Display | 128MB video memory |
| Guest Additions | Installed (clipboard sharing, screen resize) |

---

## Why Two Network Adapters?

Kali needs internet access to download tools, update packages, and access TryHackMe.
But it also needs to communicate with Metasploitable on the isolated Host-Only network.

Two adapters solve both needs cleanly:
- **Adapter 1 (NAT):** Routes through host machine. Kali gets internet. Used for `apt update`, downloading wordlists, accessing THM VPN.
- **Adapter 2 (Host-Only):** Private network between Kali and Metasploitable only. No internet routing. Attack traffic stays fully contained here.

This is a deliberate design choice — it mirrors how SOC environments segment attack surfaces.

---

## Network Configuration

**Host-Only IP (Kali):** `192.168.56.101` (assigned by VirtualBox DHCP)
**Host-Only IP (Metasploitable):** `192.168.56.102`

To verify your IPs inside Kali:
```bash
ip addr show
```

To test connectivity to Metasploitable:
```bash
ping 192.168.56.102
```

---

## Tools Pre-installed on Kali (Used in This Lab)

Kali comes with most tools by default. The following are actively used in this lab:

| Tool | Version Check Command | Notes |
|---|---|---|
| Metasploit Framework | `msfconsole --version` | Pre-installed on Kali |
| Nmap | `nmap --version` | Pre-installed |
| Wireshark | `wireshark --version` | Pre-installed |
| Tcpdump | `tcpdump --version` | Pre-installed |
| SQLmap | `sqlmap --version` | Pre-installed |
| Burp Suite Community | `burpsuite` | Pre-installed |
| Zenmap | `zenmap` | GUI frontend for Nmap |

---

## Splunk Installation on Kali

Splunk Free is installed directly on Kali to act as the lab SIEM.

> See [`../tools-installed/splunk-setup.md`](../tools-installed/splunk-setup.md) for full installation and configuration notes.

**Splunk dashboard access:** `http://localhost:8000` (from inside Kali browser)
**Also accessible from host machine at:** `http://192.168.56.101:8000`

---

## Common Issues Hit During Setup

**Issue 1 — Kali could not reach Metasploitable after adding Host-Only adapter**
- Root cause: The Host-Only network interface wasn't brought up automatically
- Fix: Ran `sudo dhclient eth1` to request an IP on the second adapter. Also added it to `/etc/network/interfaces` to persist on reboot.

**Issue 2 — VirtualBox Host-Only network didn't exist by default**
- Root cause: VirtualBox doesn't create a Host-Only network automatically on Windows
- Fix: VirtualBox → File → Host Network Manager → Create → set to `192.168.56.0/24` with DHCP enabled

**Issue 3 — Screen resolution too small after Kali install**
- Fix: Installed VirtualBox Guest Additions via `sudo apt install virtualbox-guest-x11` then rebooted

---

## Screenshots

> Add the following screenshots to `../screenshots/` folder:
> - `kali-desktop.png` — Kali desktop showing VM is running
> - `kali-network-config.png` — Output of `ip addr show` showing both adapters
> - `kali-splunk-dashboard.png` — Splunk home dashboard running on Kali
