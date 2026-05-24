# Home Lab — Notes & Config

## Lab Status
Last Updated: May 24, 2026

---

## Experiment Tracker

| Exp | Title | Status | Date |
|---|---|---|---|
| Exp001 | pfSense Firewall Rules | ✅ Complete | 2026-05-23 |
| Exp002 | SSH Hardening | ✅ Complete | 2026-05-23 |
| Exp003 | fail2ban SSH Intrusion Prevention | ✅ Complete | 2026-05-24 |
| Exp004 | Splunk SIEM Log Ingestion | ✅ Complete | 2026-05-24 |
| Exp005 | Nessus Vulnerability Scan | ✅ Complete | 2026-05-24 |

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
| DHCP Pool | 192.168.56.100 - 192.168.56.200 | Dynamic |

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

## Splunk (Windows Host)

- Version: Splunk Enterprise 10.4.0
- Web UI: http://localhost:8000
- Receiving port: 9997
- Credentials: [see Bitwarden — "Splunk Enterprise - Home Lab"]

---

## ISOs
- ubuntu-24.04.4-live-server-amd64.iso
- netgate-installer-v1.2-RELEASE-amd64.iso

---

## Phase Progress
- [x] Phase 1 — VirtualBox Setup
- [x] Phase 2 — Ubuntu Server VM
- [x] Phase 3 — pfSense Firewall VM
- [x] Phase 4 — Experiments
- [x] Phase 5 — GitHub Documentation

---

## Defense Layers

| Layer | Tool | What It Does |
|---|---|---|
| 1 | pfSense Firewall (Exp001) | Default deny, specific allow rules |
| 2 | SSH Hardening (Exp002) | Port 2222, key auth only, no root, no passwords |
| 3 | fail2ban (Exp003) | Dynamic blocking — bans IPs after 3 failed attempts |
| 4 | Splunk SIEM (Exp004) | Real-time log ingestion, dashboards, and alerts |
| 5 | Nessus (Exp005) | Vulnerability scanning — validates attack surface |
