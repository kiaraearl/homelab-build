# Home Lab — Notes & Config

## Lab Status
Last Updated: May 31, 2026

---

## Experiment Tracker

| Exp | Title | Status | Date |
|---|---|---|---|
| Exp001 | pfSense Firewall Rules | ✅ Complete | 2026-05-23 |
| Exp002 | SSH Hardening | ✅ Complete | 2026-05-23 |
| Exp003 | fail2ban SSH Intrusion Prevention | ✅ Complete | 2026-05-24 |
| Exp004 | Splunk SIEM Log Ingestion | ✅ Complete | 2026-05-24 |
| Exp005 | Nessus Vulnerability Scan | ✅ Complete | 2026-05-24 |
| Exp006 | Active Directory Domain Services | ✅ Complete | 2026-05-25 |
| Exp007 | Microsoft Azure + Sentinel (Cloud SIEM) | ✅ Complete | 2026-05-31 |

Full write-ups in `/experiments/`

---

## Host Machine

- OS: Windows 11 Pro
- CPU: Intel i7-12700K
- RAM: 32GB DDR4
- Storage: Samsung 990 Pro 1TB NVMe
- VM Folder: C:\Users\Kimea\VirtualBox VMs\
- ISO Folder: C:\Users\Kimea\VM-ISOs\

---

## Network Configuration

### Host-Only Network
- Range: 192.168.56.0/24
- Gateway: 192.168.56.1

### VM IP Assignments
| VM | IP | Type |
|---|---|---|
| Ubuntu-Server-01 | 192.168.56.10 | Static (enp0s8) |
| pfSense LAN | 192.168.56.2 | Static |
| WinServer2022-DC01 | 192.168.56.20 | Static (Ethernet 2) |
| DHCP Pool | 192.168.56.100 - 192.168.56.200 | Dynamic |

---

## Shared Folder — Host ↔ VMs

| Host Path | VM | VM Drive | Notes |
|---|---|---|---|
| C:\VMShare | WinServer2022-DC01 | Z:\ | Auto-mount, permanent, bidirectional |

Setup: VirtualBox → Devices → Shared Folders → Add → C:\VMShare, Auto-mount, Make Permanent
Requires: VirtualBox Guest Additions installed on VM

---

## VM 001 — Ubuntu Server

### Specs
- Name: Ubuntu-Server-01
- OS: Ubuntu Server 24.04.4 LTS
- RAM: 2GB
- Storage: 20GB VDI (dynamically allocated)
- Adapter 1 (enp0s3): NAT — 10.0.2.15
- Adapter 2 (enp0s8): Host-Only — 192.168.56.10 (static)

### Credentials
- Username: kearl
- Password: [see Bitwarden]

### SSH Access
```powershell
ssh -p 2222 -i C:\Users\Kimea\.ssh\id_ed25519 kearl@192.168.56.10
```

### SSH Keys
| Key | Purpose | Passphrase |
|---|---|---|
| id_ed25519 | Personal admin access | Yes |
| id_rsa_nessus | Nessus scanner service account | No |

### Netplan Config
- File: /etc/netplan/50-cloud-init.yaml
- enp0s3: DHCP (NAT/internet)
- enp0s8: Static 192.168.56.10/24

### Status
- [x] OS installed
- [x] SSH configured and working
- [x] Static IP assigned (192.168.56.10)
- [x] User accounts created
- [x] UFW firewall enabled
- [x] SSH hardened — port 2222, key auth only (see Exp002)
- [x] fail2ban installed and active (see Exp003)
- [x] Splunk Universal Forwarder installed (see Exp004)
- [x] Nessus credentialed scan validated (see Exp005)
- [x] All updates applied (apt update && apt upgrade)

---

## VM 002 — pfSense Firewall

### Specs
- Name: pfSense-Firewall
- OS: pfSense CE 2.8.1-RELEASE (FreeBSD/amd64)
- RAM: 1GB
- Storage: 10GB VDI
- Adapter 1 (em0): NAT — WAN — 10.0.2.15 (DHCP)
- Adapter 2 (em1): Host-Only — LAN — 192.168.56.2 (static)

### Credentials
- Web UI: http://192.168.56.2
- Username: admin
- Password: [see Bitwarden]

### DHCP Server
- Enabled on LAN
- Range: 192.168.56.100 - 192.168.56.200

### Status
- [x] ISO downloaded and verified
- [x] VM created in VirtualBox
- [x] pfSense CE 2.8.1 installed
- [x] WAN/LAN interfaces configured
- [x] LAN IP set to 192.168.56.2
- [x] DHCP server enabled on LAN
- [x] Web UI accessible from host browser
- [x] Default password changed
- [x] Firewall rules hardened (see Exp001)

---

## VM 003 — Windows Server 2022 Domain Controller

### Specs
- Name: WinServer2022-DC01
- OS: Windows Server 2022 Standard Evaluation
- RAM: 4GB
- Storage: 50GB VDI (dynamically allocated)
- Adapter 1: NAT — 10.0.2.15
- Adapter 2 (Host-Only): 192.168.56.20 (static)

### Credentials
- Username: LAB\Administrator
- Password: [see Bitwarden — "WinServer2022-DC01 - Administrator"]
- DSRM Password: [see Bitwarden — "WinServer2022-DC01 - DSRM Password"]

### Domain Info
- Domain: lab.local
- NetBIOS: LAB
- Hostname: DC01
- DNS: 127.0.0.1 (self)
- PDC Emulator: DC01.lab.local

### Domain Users
| Name | Username | UPN | OU |
|---|---|---|---|
| Kiara Earl | kearl | kearl@lab.local | SOC_Team |
| Jane Doe | jdoe | jdoe@lab.local | SOC_Team |
| John Smith | jsmith | jsmith@lab.local | IT_Admin |

### Splunk Universal Forwarder
- Receiver: 192.168.56.1:9997
- inputs.conf: C:\Program Files\SplunkUniversalForwarder\etc\system\local\
- Sourcetype: WinEventLog:Security
- Index: main

### Azure Arc
- Arc machine name: DC01
- Resource group: RG-SOCLab
- Status: Connected
- Agent: himds (Azure Hybrid Instance Metadata Service)

### Azure Monitor Agent
- Version: 1.42.0.0
- Extension: AzureMonitorWindowsAgent (via Arc)
- Data Collection Rule: DCR-WinServer-SecurityEvents
- Ships: All Windows Security Events → LAW-SOCLab

### VirtualBox Guest Additions
- Installed: Yes
- Shared folder: Z:\ → C:\VMShare (host)
- Clipboard: Bidirectional
- Drag and drop: Bidirectional

### Status
- [x] OS installed (Windows Server 2022 Standard Evaluation)
- [x] Static IP assigned (192.168.56.20)
- [x] Renamed to DC01
- [x] AD DS role installed
- [x] Promoted to Domain Controller
- [x] Domain lab.local created
- [x] OUs created (SOC_Team, IT_Admin, Workstations)
- [x] Domain users created
- [x] Splunk Universal Forwarder installed and shipping Security logs
- [x] VirtualBox Guest Additions installed
- [x] Shared folder configured (Z:\ → C:\VMShare)
- [x] Azure Arc agent installed (Connected)
- [x] Azure Monitor Agent deployed (v1.42.0.0)
- [x] Windows Security Events flowing to LAW-SOCLab

---

## Splunk (Windows Host)

- Version: Splunk Enterprise 10.4.0
- Web UI: http://localhost:8000
- Receiving port: 9997
- Credentials: [see Bitwarden — "Splunk Enterprise - Home Lab"]

---

## Azure (Cloud)

| Resource | Value |
|---|---|
| Account | Kimearls24@outlook.com |
| Subscription | Azure subscription 1 |
| Subscription ID | 06479c4b-a951-43d0-905c-e0b101ae740a |
| Resource Group | RG-SOCLab |
| Region | East US |
| Log Analytics Workspace | LAW-SOCLab |
| Sentinel | Enabled (free trial through 7/1/2026, 10 GB/day) |
| Arc Machine | DC01 (WinServer2022-DC01) |
| Data Connector | Windows Security Events via AMA |
| Data Collection Rule | DCR-WinServer-SecurityEvents |
| Analytics Rule | Brute Force - Multiple Failed Logons (High, T1110) |

### Sentinel Saved Queries
| Query Name | Description |
|---|---|
| Failed Logon Attempts - 4625 | Hunts for EventID 4625 — failed logons |
| Brute Force Summary by Account | Summarizes failed logon count by account and computer |

---

## ISOs
- ubuntu-24.04.4-live-server-amd64.iso
- netgate-installer-v1.2-RELEASE-amd64.iso
- SERVER_EVAL_x64FRE_en-us.iso (Windows Server 2022 Standard Evaluation)

---

## Phase Progress
- [x] Phase 1 — VirtualBox Setup
- [x] Phase 2 — Ubuntu Server VM
- [x] Phase 3 — pfSense Firewall VM
- [x] Phase 4 — Experiments
- [x] Phase 5 — GitHub Documentation
- [x] Phase 6 — Cloud SIEM (Azure + Sentinel)

---

## Defense Layers

| Layer | Tool | What It Does |
|---|---|---|
| 1 | pfSense Firewall (Exp001) | Default deny, specific allow rules |
| 2 | SSH Hardening (Exp002) | Port 2222, key auth only, no root, no passwords |
| 3 | fail2ban (Exp003) | Dynamic blocking — bans IPs after 3 failed attempts |
| 4 | Splunk SIEM (Exp004) | Real-time log ingestion, dashboards, and alerts |
| 5 | Nessus (Exp005) | Vulnerability scanning — validates attack surface |
| 6 | Active Directory (Exp006) | Identity & access management, AD event monitoring in Splunk |
| 7 | Azure Sentinel (Exp007) | Cloud SIEM, hybrid log ingestion, KQL hunting, automated detection |

---

## Troubleshooting Reference

### Azure Monitor Agent — Arc Extension on VirtualBox
- Arc extension may report "Succeeded" in portal but AzureMonitorAgent service won't register on VM
- This is a known timing issue with VirtualBox/NAT and Arc extension deployment
- Verify Arc connectivity: `azcmagent check` — all endpoints should show Reachable: true
- Verify extension files landed: `Get-ChildItem "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent"`
- Manual MSI install NOT supported on Windows Server — must use Arc extension
- If extension fails repeatedly: delete extension in portal, wait 5 min, re-add via Sentinel Data Connectors → Windows Security Events via AMA → Create DCR (this triggers extension install through the proper Sentinel pathway)

### PowerShell Script Execution from Shared Folder (Z:\)
Scripts on network/mapped drives blocked by default execution policy. Fix before running any script from Z:\:
```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
.\scriptname.ps1
```

### WinEventLog Monitoring — Universal Forwarder on Windows
CLI command `splunk add monitor WinEventLog://Security` does NOT work on Windows Universal Forwarder.
Correct method — create inputs.conf at:
`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`
```
[WinEventLog://Security]
index = main
disabled = false
```
