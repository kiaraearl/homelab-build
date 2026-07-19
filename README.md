# Home Lab Build — Documentation

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
- [x] Deploy a cloud SIEM and connect on-premises infrastructure via hybrid cloud
- [x] Deploy a DNS sinkhole for network-wide threat blocking
- [x] Build and troubleshoot a domain-joined Windows 11 endpoint
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
| **VirtualBox** | Primary hypervisor for all VMs | Complete |
| **pfSense CE** | Firewall / router VM — network segmentation | Complete |
| **Ubuntu Server 24.04** | Linux server + desktop environment | Complete |
| **Windows Server 2022** | Active Directory Domain Controller — lab.local | Complete |
| **Windows 11 Pro** | Domain-joined endpoint — troubleshooting labs | Complete |
| **Pi-hole (Ubuntu Server)** | DNS sinkhole — network-wide ad/threat blocking | Complete |
| **Kali Linux** | Offensive security tooling practice | Future |

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
         |
    [Win11-Workstation-01 WS01]  ←  192.168.56.30 (domain-joined endpoint)
         |
    [Pihole-01]  ←  192.168.56.40 (DNS sinkhole)
         |
    [Azure Cloud]  ←  DC01 Arc-connected → LAW-SOCLab → Microsoft Sentinel
```

### Network Config
- **Host-Only Network:** `192.168.56.0/24`
- **DHCP Pool:** `192.168.56.100 - 192.168.56.200` (managed by pfSense)
- **Ubuntu SSH:** Port 2222, key-based auth only
- **Shared Folder:** `C:\VMShare` (host) ↔ `Z:\` (WinServer2022-DC01)
- **DNS:** All lab devices point to Pi-hole (192.168.56.40) for DNS filtering

---

## VM Build Log

### VM 001 — Ubuntu Server
**Purpose:** Linux CLI + desktop practice, security tooling, log generation

| Setting | Value |
|---|---|
| OS | Ubuntu Server 24.04.4 LTS + GNOME Desktop |
| RAM | 4GB |
| Storage | 40GB dynamically allocated |
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
- [x] Ubuntu Desktop (GNOME) installed
- [x] Disk expanded to 40GB

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

### VM 003 — Windows Server 2022 Domain Controller
**Purpose:** Active Directory, identity management, Windows event log generation, Azure Arc endpoint

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
- [x] VirtualBox Guest Additions installed
- [x] Shared folder configured (Z:\ → C:\VMShare)
- [x] Azure Arc agent installed — DC01 registered as Arc-enabled server
- [x] Azure Monitor Agent deployed — Windows Security Events flowing to Sentinel

---

### VM 004 — Windows 11 Workstation
**Purpose:** Domain-joined Windows endpoint for help desk and troubleshooting labs

| Setting | Value |
|---|---|
| OS | Windows 11 Pro |
| RAM | 4GB |
| Storage | 60GB dynamically allocated |
| IP | 192.168.56.30 (static) |
| Hostname | WS01 |
| Domain | lab.local |
| DNS | 192.168.56.40 (Pi-hole) |

**Progress:**
- [x] Windows 11 Pro installed (TPM bypass via LabConfig registry key)
- [x] Static IP assigned (192.168.56.30)
- [x] Joined to lab.local domain
- [x] Verified in ADUC on DC01 (WS01.lab.local)
- [x] DNS pointed to Pi-hole

---

### VM 005 — Pi-hole DNS Sinkhole
**Purpose:** Network-wide DNS filtering, ad/tracking/malware domain blocking, DNS traffic visibility

| Setting | Value |
|---|---|
| OS | Ubuntu Server 24.04.4 LTS |
| RAM | 1GB |
| Storage | 10GB |
| IP | 192.168.56.40 (static) |
| Dashboard | http://192.168.56.40/admin |
| Upstream DNS | Cloudflare (1.1.1.1) with DNSSEC |
| Blocklist | StevenBlack's Unified Hosts List (81,382 domains) |

**Progress:**
- [x] Ubuntu Server installed
- [x] Static IP assigned (192.168.56.40)
- [x] Pi-hole installed and active
- [x] 81,382 domains on blocklist
- [x] Query logging enabled
- [x] WS01 and host DNS pointed to Pi-hole
- [x] YouTube domains whitelisted

---

## Security Tooling

| Tool | Purpose | Status |
|---|---|---|
| **pfSense** | Firewall — default deny ruleset | Complete |
| **fail2ban** | SSH brute force prevention | Complete |
| **Splunk Enterprise** | SIEM — log ingestion, dashboards, alerts | Complete |
| **Nessus Essentials Plus** | Vulnerability scanning | Complete |
| **Active Directory** | Identity & access management, Windows event monitoring | Complete |
| **Microsoft Sentinel** | Cloud SIEM — KQL hunting, analytics rules, incident management | Complete |
| **Azure Arc** | Hybrid cloud — on-prem server management from Azure | Complete |
| **Pi-hole** | DNS sinkhole — network-wide ad and malicious domain blocking | Complete |
| **Wazuh** | EDR — FIM, brute-force detection, Active Response, vulnerability scanning | Complete |
| **Okta** | IdP / SSO — SAML dev tenant configuration | Complete |
| **Zendesk** | Support ticketing — SLA policy, escalation automation | Complete |
| **Wireshark** | Network traffic analysis | Future |
| **Snort / Suricata** | IDS/IPS practice | Future |
| **Kali Linux** | Offensive security tooling | Future |

---

## Experiments & Write-Ups

| # | Experiment | Status | Write-Up |
|---|---|---|---|
| 001 | pfSense firewall rule configuration — default deny ruleset | Complete | [View](experiments/Exp001-pfSense-Firewall-Rules/exp001-pfsense-firewall-rules.md) |
| 002 | SSH hardening — port 2222, key auth, disable root & passwords | Complete | [View](experiments/Exp002-SSH-Hardening/exp002-ssh-hardening.md) |
| 003 | fail2ban — SSH intrusion prevention, live ban test | Complete | [View](experiments/Exp003-Fail2Ban-Intrusion-Prevention/exp003-fail2ban.md) |
| 004 | Splunk SIEM — log ingestion, dashboard, real-time alert | Complete | [View](experiments/Exp004-Splunk-SIEM/exp004-splunk-siem.md) |
| 005 | Nessus — unauthenticated + credentialed vulnerability scan | Complete | [View](experiments/Exp005-Nessus-Vulnerability-Scan/exp005-nessus-vulnerability-scan.md) |
| 006 | Active Directory — Windows Server 2022, lab.local domain, AD DS, Splunk integration | Complete | [View](experiments/Exp006-Active-Directory/exp006-active-directory.md) |
| 007 | Microsoft Azure + Sentinel — cloud SIEM, Azure Arc, KQL hunting, brute force detection | Complete | [View](experiments/Exp007-Azure-Sentinel/exp007-azure-sentinel.md) |
| 008 | Pi-hole DNS Sinkhole — network-wide DNS filtering, 81k+ domain blocklist, live query monitoring | Complete | [View](experiments/Exp008-Pihole-DNS-Sinkhole/exp008-pihole-dns-sinkhole.md) |
| 009 | CIC Incident Operations — SITREP, break/fix runbook, exec summary, ServiceNow lifecycle | Complete | [View](experiments/Exp009-CIC-Incident-Operations/exp009-cic-incident-ops-sitrep-template.md) |
| 010 | Wazuh EDR — Linux/Windows agent deployment, FIM, brute-force detection, Active Response, vulnerability scanning | Complete | [View](experiments/Exp010-Wazuh-EDR/exp010-wazuh-edr.md) |
| 011 | SSO/SAML Dev Tenant Demo (Okta) — IdP-side SSO configuration | Complete | [View](experiments/Exp011-SSO-SAML-Okta/exp011-sso-saml.md) |
| 012 | IAM Access Request Workflow — on-prem AD access request lifecycle | Complete | [View](experiments/Exp012-IAM-Access-Workflow/exp012-iam-access-workflow.md) |
| 013 | Zendesk Support Operations Lab — ticketing, SLA policy, escalation automation | Complete | [View](experiments/Exp013-Zendesk-Support-Operations/exp013-Zendesk-Support-Operations-Lab.md) |

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
| 7 | Microsoft Sentinel | Cloud SIEM — KQL detection, MITRE-mapped analytics rules, incident queue |
| 8 | Pi-hole DNS Sinkhole | Blocks 81,382 known ad, tracking, and malicious domains at DNS level |
| 9 | Wazuh EDR | Endpoint detection & response — FIM, brute-force detection, Active Response |

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
| DNS filtering & sinkhole (Pi-hole) | Network+ / Security+ / CySA+ |
| Windows 11 endpoint troubleshooting | A+ Core 2 / Help Desk |
| Azure cloud fundamentals | AZ-900 |
| Cloud security concepts | SC-900 |
| Sentinel analytics rules, KQL, incident management | SC-200 |
| Endpoint detection & response (Wazuh) | Security+ / CySA+ |
| Federated identity, SSO/SAML (Okta) | Security+ / Help Desk |
| IAM access request lifecycle | Security+ / Help Desk |
| Ticketing, SLA management, escalation triage (Zendesk) | Help Desk / SOC |

---

## Repository Structure

```
homelab-build/
├── README.md
├── lab-notes.md                          ← VM specs, network config, credentials reference
├── experiments/
│   ├── Exp001-pfSense-Firewall-Rules/
│   │   ├── exp001-pfsense-firewall-rules.md
│   │   └── images/
│   ├── Exp002-SSH-Hardening/
│   │   ├── exp002-ssh-hardening.md
│   │   └── images/
│   ├── Exp003-Fail2Ban-Intrusion-Prevention/
│   │   ├── exp003-fail2ban.md
│   │   └── images/
│   ├── Exp004-Splunk-SIEM/
│   │   ├── exp004-splunk-siem.md
│   │   └── images/
│   ├── Exp005-Nessus-Vulnerability-Scan/
│   │   ├── exp005-nessus-vulnerability-scan.md
│   │   └── images/
│   ├── Exp006-Active-Directory/
│   │   ├── exp006-active-directory.md
│   │   └── Setup Images/
│   │       ├── Config/
│   │       ├── Network Config/
│   │       └── Verification/
│   ├── Exp007-Azure-Sentinel/
│   │   ├── exp007-azure-sentinel.md
│   │   └── images/
│   ├── Exp008-Pihole-DNS-Sinkhole/
│   │   ├── exp008-pihole-dns-sinkhole.md
│   │   └── images/
│   ├── Exp009-CIC-Incident-Operations/
│   │   ├── exp009-cic-incident-ops-exec-summary.md
│   │   ├── exp009-cic-incident-ops-runbook-fail2ban-restart.md
│   │   ├── exp009-cic-incident-ops-sitrep-template.md
│   │   └── images/
│   ├── Exp010-Wazuh-EDR/
│   │   ├── exp010-wazuh-edr.md
│   │   └── images/
│   ├── Exp011-SSO-SAML-Okta/
│   │   ├── exp011-sso-saml.md
│   │   └── images/
│   ├── Exp012-IAM-Access-Workflow/
│   │   ├── exp012-iam-access-workflow.md
│   │   └── images/
│   ├── Exp013-Zendesk-Support-Operations/
│   │   ├── exp013-Zendesk-Support-Operations-Lab.md
│   │   └── images/
│   └── setup/                            ← initial Ubuntu host setup screenshots
├── labs/
│   └── Lab01-win11-setup/
│       └── Images/
│           ├── setup/
│           └── verification/
└── incident-reports/
    └── IR-2026-001-SSH-BruteForce.pdf
```

---

## Author

**Kiara Earl**
CompTIA A+ Certified | WGU B.S. Cybersecurity & Information Assurance (Expected 2027)
kimearls24@outlook.com
Houston, TX
[Portfolio](https://kiaraearl.github.io) | [GitHub](https://github.com/kiaraearl)

---

> *Built from scratch. Documented from day one.*
