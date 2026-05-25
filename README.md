# Home Lab Build — Documentation 🖥️

> Ongoing documentation of my personal cybersecurity home lab —
> hardware specs, VM setup, network configuration, and security experiments.
>
> **Kiara Earl** — CompTIA A+ Certified | WGU B.S. Cybersecurity Student

---

## Why I Built This

A home lab is the single best way to build hands-on IT and cybersecurity skills outside
of a job. This repository documents everything — hardware decisions, OS installations,
network configuration, and security experiments — so the learning is visible,
reproducible, and portfolio-ready.

---

## Lab Goals

- [x] Practice OS installation, configuration, and troubleshooting
- [x] Simulate enterprise networking (firewall rules, DHCP, DNS)
- [x] Run vulnerability assessments in a safe, isolated environment
- [x] Build a functional SOC-like detection and monitoring setup
- [x] Support Network+ and Security+ hands-on study

---

## Host Machine

| Component | Details |
|---|---|
| **CPU** | Intel Core i7-12700K |
| **Motherboard** | MSI PRO B760-P WiFi DDR4 (ATX) |
| **RAM** | 32GB DDR4 (2x16GB dual-channel) |
| **Storage** | Samsung 990 Pro 1TB NVMe |
| **GPU** | AMD Radeon RX 7900 XT |
| **OS** | Windows 11 Pro |
| **Case** | Hyte Y70 ATX (Pink) |

> Full build documentation: [Custom PC Build Repo](https://github.com/kiaraearl/pc-build)

---

## Virtualization Stack

| Tool | Purpose | Status |
|---|---|---|
| **VirtualBox** | Primary hypervisor for all VMs | ✅ Complete |
| **pfSense CE** | Firewall / router VM — network segmentation | ✅ Complete |
| **Ubuntu Server 24.04** | Linux practice environment | ✅ Complete |
| **Windows Server 2022** | Active Directory Domain Controller — lab.local | ✅ Complete |
| **Kali Linux** | Offensive security tooling practice | ⏳ Future |

---

## Network Design

```
[VirtualBox Host — Windows 11 Pro — 192.168.56.1]
         |
    [pfSense VM]  ←  firewall / LAN: 192.168.56.2
         |
    [Ubuntu Server VM]  ←  192.168.56.10
         |
    [WinServer2022-DC01]  ←  192.168.56.20 (Domain Controller — lab.local)
```

### Network Config
- **Host-Only Network:** `192.168.56.0/24`
- **DHCP Pool:** `192.168.56.100 - 192.168.56.200` (managed by pfSense)
- **Ubuntu SSH:** Port 2222, key-based auth only

---

## VM Build Log

### VM 001 — Ubuntu Server
**Purpose:** Linux CLI practice, security tooling, log generation

| Setting | Value |
|---|---|
| OS | Ubuntu Server 24.04.4 LTS |
| RAM | 2GB |
| Storage | 20GB dynamically allocated |
| IP | 192.168.56.10 (static) |
| SSH | Port 2222, ed25519 key auth |

**Progress:**
- [x] OS installed
- [x] SSH configured and hardened (port 2222, key auth only)
- [x] Static IP assigned
- [x] UFW firewall enabled
- [x] fail2ban installed and active
- [x] Splunk Universal Forwarder installed
- [x] Nessus credentialed scan validated

---

### VM 003 — Windows Server 2022 Domain Controller
**Purpose:** Active Directory, identity management, Windows event log generation

| Setting | Value |
|---|---|
| OS | Windows Server 2022 Standard Evaluation |
| RAM | 4GB |
| Storage | 50GB dynamically allocated |
| IP | 192.168.56.20 (static) |
| Domain | lab.local |
| Hostname | DC01 |

**Progress:**
- [x] OS installed
- [x] Static IP assigned (192.168.56.20)
- [x] Renamed to DC01
- [x] AD DS role installed and configured
- [x] Promoted to Domain Controller (lab.local)
- [x] OUs created (SOC_Team, IT_Admin, Workstations)
- [x] Domain users created (kearl, jdoe, jsmith)
- [x] Splunk Universal Forwarder shipping Security logs to SIEM

---

### VM 002 — pfSense Firewall
**Purpose:** Network segmentation, firewall rules, DHCP management

| Setting | Value |
|---|---|
| OS | pfSense CE 2.8.1 |
| RAM | 1GB |
| Storage | 10GB |
| LAN IP | 192.168.56.2 (static) |

**Progress:**
- [x] OS installed
- [x] WAN/LAN interfaces configured
- [x] DHCP server enabled on LAN
- [x] Web UI accessible from host
- [x] Default deny firewall ruleset configured

---

## Security Tooling

| Tool | Purpose | Status |
|---|---|---|
| **pfSense** | Firewall — default deny ruleset | ✅ Complete |
| **fail2ban** | SSH brute force prevention | ✅ Complete |
| **Splunk Enterprise** | SIEM — log ingestion, dashboards, alerts | ✅ Complete |
| **Nessus Essentials Plus** | Vulnerability scanning | ✅ Complete |
| **Active Directory** | Identity & access management, Windows event monitoring | ✅ Complete |
| **Wireshark** | Network traffic analysis | ⏳ Future |
| **Snort / Suricata** | IDS/IPS practice | ⏳ Future |

---

## Experiments & Write-Ups

| # | Experiment | Status | Write-Up |
|---|---|---|---|
| 001 | pfSense firewall rule configuration — default deny ruleset | ✅ Complete | [View](experiments/exp001-pfsense-firewall-rules.md) |
| 002 | SSH hardening — port 2222, key auth, disable root & passwords | ✅ Complete | [View](experiments/exp002-ssh-hardening.md) |
| 003 | fail2ban — SSH intrusion prevention, live ban test | ✅ Complete | [View](experiments/exp003-fail2ban.md) |
| 004 | Splunk SIEM — log ingestion, dashboard, real-time alert | ✅ Complete | [View](experiments/exp004-splunk-siem.md) |
| 005 | Nessus — unauthenticated + credentialed vulnerability scan | ✅ Complete | [View](experiments/exp005-nessus-vulnerability-scan.md) |
| 006 | Active Directory — Windows Server 2022, lab.local domain, AD DS, Splunk integration | ✅ Complete | [View](experiments/exp006-active-directory.md) |

## Incident Reports

| ID | Title | Format |
|---|---|---|
| IR-2026-001 | SSH Brute Force Attack — Ubuntu-Server-01 | [PDF](incident-reports/IR-2026-001-SSH-BruteForce.pdf) |
---

## Defense Layers Built

| Layer | Tool | What It Does |
|---|---|---|
| 1 | pfSense Firewall | Default deny — only port 2222 allowed inbound |
| 2 | SSH Hardening | Port 2222, key auth only, no root, no passwords |
| 3 | fail2ban | Dynamic blocking — bans IPs after 3 failed attempts |
| 4 | Splunk SIEM | Real-time log ingestion, dashboards, and alerts |
| 5 | Nessus | Vulnerability scanning — validates attack surface |
| 6 | Active Directory | Identity & access management, AD event monitoring in Splunk |

---

## Cert Connections

| Lab Activity | Cert Relevance |
|---|---|
| Hardware documentation | A+ Core 1 |
| OS installation & config | A+ Core 2 |
| VM networking & subnetting | Network+ |
| Firewall rules & segmentation | Network+ / Security+ |
| SSH hardening | Security+ |
| Intrusion prevention (fail2ban) | Security+ / CySA+ |
| SIEM log ingestion & alerting | Security+ / CySA+ |
| Vulnerability scanning (Nessus) | Security+ / CySA+ |
| Active Directory & identity management | Security+ / CySA+ |

---

## Repository Structure

```
homelab-build/
├── README.md
├── lab-notes.md              ← VM specs, network config, credentials reference
├── experiments/
│   ├── exp001-pfsense-firewall-rules.md
│   ├── exp002-ssh-hardening.md
│   ├── exp003-fail2ban.md
│   ├── exp004-splunk-siem.md
│   ├── exp005-nessus-vulnerability-scan.md
│   ├── exp006-active-directory.md
│   └── images/
│       ├── exp001/
│       ├── exp002/
│       ├── exp003/
│       ├── exp004/
│       ├── exp005/
│       └── exp006/
│           ├── setup/
│           ├── config/
│           └── verification/
```

---

## Author

**Kiara Earl**
CompTIA A+ Certified | WGU B.S. Cybersecurity & Information Assurance (Expected 2027)
📧 kimearls24@outlook.com
📍 Houston, TX
🔗 [Portfolio](https://kiaraearl.github.io) | [GitHub](https://github.com/kiaraearl)

---

> *Built from scratch. Documented from day one.*
