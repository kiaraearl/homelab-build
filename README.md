# Home Lab Build — Documentation 🖥️

> Ongoing documentation of my personal cybersecurity home lab —
> hardware specs, VM setup, network configuration, and security tooling.
>
> Authored by Kiara Earl — CompTIA A+ Certified | WGU Cybersecurity Student

---

## Why I Built This

A home lab is the single best way to build hands-on IT and cybersecurity skills outside
of a job. This repository documents everything — hardware decisions, OS installations,
network configuration, and security experiments — so the learning is visible,
reproducible, and portfolio-ready.

---

## Lab Goals

- [x] Practice OS installation, configuration, and troubleshooting
- [x] Simulate enterprise networking (VLANs, DHCP, DNS, firewalls)
- [ ] Run vulnerability assessments in a safe, isolated environment
- [ ] Build a functional SOC-like detection and monitoring setup
- [ ] Support Network+ and Security+ hands-on study

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
| **pfSense** | Firewall / router VM — network segmentation | ✅ Complete |
| **Ubuntu Server** | Linux practice environment | ✅ Complete |
| **Windows Server** | Active Directory and Group Policy practice | ⏳ Future |
| **Kali Linux** | Offensive security tooling practice | ⏳ Future |

---

## Network Design

```
[ISP Modem]
     |
[Home Router]
     |
[VirtualBox Host — Windows 11 Pro]
     |
  [pfSense VM] ← firewall / segmentation
     |
  ┌──────────────┐
  │              │
[Ubuntu VM]  [Windows Server VM]
(Linux lab)  (AD / GPO practice)
```

### Planned Network Config
- **LAN Segment:** `192.168.56.0/24` — primary lab network
- **DHCP:** Managed by pfSense VM
- **DNS:** pfSense with upstream forwarding
- **VLAN Segmentation:** Isolate attack VM from rest of lab

---

## VM Build Log

### VM 001 — Ubuntu Server
**Purpose:** Linux CLI practice, hosting practice, scripting

| Setting | Value |
|---|---|
| RAM | 2GB |
| Storage | 20GB dynamically allocated |
| Network | Internal Network (VirtualBox) |
| OS | Ubuntu Server 24.04 LTS |

**Progress:**
- [x] OS installed
- [x] SSH configured
- [x] Static IP assigned
- [x] User accounts created
- [x] Basic firewall (UFW) configured

---

### VM 002 — pfSense Firewall
**Purpose:** Network segmentation, firewall rules, DHCP/DNS management

| Setting | Value |
|---|---|
| RAM | 1GB |
| Storage | 10GB |
| Network Adapters | WAN (NAT) + LAN (Internal Network) |
| OS | pfSense CE |

**Progress:**
- [x] ISO downloaded and verified
- [x] VM created in VirtualBox
- [x] pfSense installed
- [x] WAN/LAN interfaces configured
- [x] DHCP server enabled on LAN
- [x] Web UI accessible from host

---

## Security Tooling (Planned)

| Tool | Purpose | Status |
|---|---|---|
| **Nessus Essentials** | Vulnerability scanning (free tier) | ⏳ Planned |
| **OpenVAS** | Open source vulnerability assessment | ⏳ Planned |
| **Wireshark** | Network traffic analysis | ⏳ Planned |
| **Snort / Suricata** | IDS/IPS practice | ⏳ Future |
| **Splunk Free** | Log aggregation and SIEM practice | ⏳ Future |

---

## Experiments & Write-Ups

| # | Experiment | Status |
|---|---|---|
| 001 | pfSense firewall rule configuration | ✅ Complete |
| 002 | Ubuntu SSH hardening (key auth, port change, disable root) | ✅ Complete |
| 003 | Nessus scan against Ubuntu VM | ⏳ Planned |
| 004 | VLAN segmentation in VirtualBox | ⏳ Planned |
| 005 | Simulate and detect a port scan (Nmap) | ⏳ Future |

---

## Resources I'm Using

- Professor Messer's Network+ Course (Free)
- TryHackMe — SOC Level 1 Path
- Cisco Packet Tracer (SOHO network simulation)
- VirtualBox Documentation
- pfSense Official Docs

---

## Cert Connections

| Lab Activity | Cert Relevance |
|---|---|
| Hardware documentation | A+ Core 1 |
| OS installation & config | A+ Core 2 |
| VM networking & subnetting | Network+ |
| Firewall rules & segmentation | Network+ / Security+ |
| Vulnerability scanning | Security+ / CySA+ |
| Log review & SIEM | CySA+ |

---

## Author

**Kiara Earl**
CompTIA A+ Certified | WGU B.S. Cybersecurity & Information Assurance (Expected 2027)
📧 kimearls24@outlook.com
📍 Houston, TX
🔗 [Portfolio](https://kiaraearl.github.io)

---

> *Built from scratch. Documented from day one.*
