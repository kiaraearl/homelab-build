# Home Lab — Notes & Config

## Lab Status
Last Updated: May 23, 2026

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
- Ubuntu Server: 192.168.56.10 (static, on enp0s8)
- pfSense LAN: 192.168.56.2 (static)
- DHCP Pool: 192.168.56.100 - 192.168.56.200

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
- From PowerShell: ssh kearl@192.168.56.10

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
- [x] OpenSSH allowed through UFW
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

---

## ISOs
- ubuntu-24.04.4-live-server-amd64.iso
- netgate-installer-v1.2-RELEASE-amd64.iso

---

## Phase Progress
- [x] Phase 1 — VirtualBox Setup
- [x] Phase 2 — Ubuntu Server VM
- [x] Phase 3 — pfSense Firewall VM
- [ ] Phase 4 — Experiments
- [ ] Phase 5 — GitHub Documentation

---

## Next Steps
- Start Experiment 001 — pfSense firewall rules
- Start Experiment 002 — SSH hardening
- Update homelab-build GitHub README checkboxes
- Push lab-notes.md to GitHub

---

## Screenshots Taken
- ubuntu-install-progress.png
- ubuntu-first-login.png
- ubuntu-ip-address.png
- ubuntu-updates-complete.png
- ubuntu-static-ip-confirmed.png
- ubuntu-ssh-connected.png
- ubuntu-ufw-status.png
- pfsense-install-welcome.png
- pfsense-interfaces.png
- pfsense-lan-config.png
- pfsense-install-complete.png
- pfsense-webui-login.png
- pfsense-dashboard.png
- pfsense-dhcp-config.png
---

## Experiment 002 — SSH Hardening

**Date:** 2026-05-23
**Status:** ✅ Complete

### Objective
Harden the SSH configuration on Ubuntu-Server-01 to reduce attack surface and eliminate password-based authentication.

### What I Did

#### 1. Modified /etc/ssh/sshd_config
Changed the following settings:

| Setting | Before | After | Reason |
|---|---|---|---|
| Port | 22 | 2222 | Avoid automated bot scans on default port |
| PermitRootLogin | prohibit-password | no | Prevent any root SSH access |
| MaxAuthTries | 6 | 3 | Limit brute force attempts |
| LoginGraceTime | 2m | 30s | Reduce attack window |
| X11Forwarding | yes | no | Server has no GUI, unnecessary exposure |
| PasswordAuthentication | yes | no | Keys only, no password login |

#### 2. Updated UFW Firewall Rules
- Opened port 2222/tcp
- Removed OpenSSH rule (port 22)
- Result: Only port 2222 accepts SSH connections

#### 3. Generated SSH Key Pair on Windows
- Algorithm: ed25519 (modern, most secure)
- Private key stays on Windows PC only
- Public key copied to server's ~/.ssh/authorized_keys

#### 4. Disabled Password Authentication
- Set PasswordAuthentication no in sshd_config
- Set KbdInteractiveAuthentication no
- Set UsePAM no
- Discovered /etc/ssh/sshd_config.d/50-cloud-init.conf was overriding settings
- Fixed override by updating 50-cloud-init.conf directly

### Verification
- Ran `sudo sshd -T | grep -E "port|permitrootlogin|maxauthtries|logingraceTime|x11forwarding"` to confirm settings loaded
- Tested password login with `-o PreferredAuthentications=password` — returned `Permission denied (publickey)` ✅
- Confirmed key-based login works with passphrase ✅

### Screenshots
- `images/ssh-hardening-verified.png` — hardening settings confirmed live
- `images/ssh-password-disabled.png` — password authentication rejected

### Key Lessons
- Ubuntu 24.04 includes /etc/ssh/sshd_config.d/ which can override main config
- UsePAM yes can bypass PasswordAuthentication no — both must be set
- Always verify changes with `sudo sshd -t` before restarting to catch config errors
- Never close your active session before confirming key auth works