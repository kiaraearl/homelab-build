# Home Lab — Notes & Config

## Lab Status
Last Updated: May 24, 2026

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
- From PowerShell: ssh -p 2222 -i C:\Users\Kimea\.ssh\id_ed25519 kearl@192.168.56.10

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
- [x] SSH hardened — port 2222, key auth only (see Experiment 002)
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
- [x] Phase 4 — Experiments
- [x] Phase 5 — GitHub Documentation

---

## Next Steps
- Exp005 — Nessus vulnerability scan
- Update homelab-build GitHub README checkboxes

---

## Experiment 001 — pfSense Firewall Rules

**Date:** 2026-05-23
**Status:** ✅ Complete

### Objective
Replace default allow-all firewall rules on pfSense with a minimal, explicit ruleset following the principle of least privilege.

### What I Did

#### Default Rules Found
- Anti-Lockout Rule (kept — protects web UI access)
- Default allow LAN to any (IPv4) — too permissive, disabled
- Default allow LAN IPv6 to any — too permissive, disabled

#### Rules Created (in order)
| Rule | Protocol | Source | Destination | Port | Action |
|---|---|---|---|---|---|
| Allow DNS | UDP | LAN subnets | 192.168.56.2 | 53 | Pass |
| Allow SSH to Ubuntu | TCP | LAN subnets | 192.168.56.10 | 2222 | Pass |
| Allow ICMP ping | ICMP | LAN subnets | Any | Any | Pass |
| Block all other traffic | Any | Any | Any | Any | Block |

#### Key Concepts Applied
- **Default deny** — block everything not explicitly allowed
- **Least privilege** — only open what's needed
- **Rule order matters** — pfSense reads top to bottom, first match wins
- Specific allow rules go at top, catch-all block goes at bottom

### Verification
- Pinged pfSense (192.168.56.2) from Ubuntu — 128/128 packets received, 0% packet loss ✅
- SSH to Ubuntu on port 2222 still works through firewall ✅

### Screenshots
- `images/exp001/exp001-01-pfsense-firewall-rules.png` — final ruleset with default rules disabled

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
- `images/exp002/exp002-01-ssh-hardening-verified.png` — hardening settings confirmed live
- `images/exp002/exp002-02-ssh-password-disabled.png` — password authentication rejected

### Key Lessons
- Ubuntu 24.04 includes /etc/ssh/sshd_config.d/ which can override main config
- UsePAM yes can bypass PasswordAuthentication no — both must be set
- Always verify changes with `sudo sshd -t` before restarting to catch config errors
- Never close your active session before confirming key auth works
---

## Experiment 003 — fail2ban SSH Intrusion Prevention

**Date:** 2026-05-24
**System:** Ubuntu-Server-01 (192.168.56.10)
**Objective:** Deploy fail2ban to automatically detect and block SSH brute force attempts

---

### What is fail2ban

fail2ban is an intrusion prevention system that watches log files in real time.
When it detects repeated failed login attempts from the same IP, it automatically
blocks that IP using the system firewall (UFW in this case).

- **UFW** = static bouncer — blocks IPs you define ahead of time
- **fail2ban** = dynamic bouncer — watches live activity and bans automatically

---

### Installation

```bash
sudo apt update && sudo apt install fail2ban -y
```

Installed packages:
- `fail2ban` — main tool
- `python3-pyinotify` — watches log files in real time for changes
- `python3-pyasyncore` — handles async I/O for log event processing
- `whois` — IP lookup support

---

### Configuration

Created local override file (never edit jail.conf directly — it gets
overwritten on updates):

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Added custom sshd jail:

```ini
[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 300
```

| Setting | Value | Meaning |
|---|---|---|
| port | 2222 | Custom SSH port from Exp002 |
| logpath | /var/log/auth.log | Ubuntu auth log fail2ban watches |
| maxretry | 3 | 3 failures triggers a ban |
| bantime | 600 | Ban lasts 10 minutes |
| findtime | 300 | Failures must occur within 5 minutes |

---

### Troubleshooting — Duplicate [sshd] Section

**Problem:** jail.conf contained a second [sshd] block further down the file.
When copied to jail.local, fail2ban threw an error on restart:

```
ERROR: While reading from '/etc/fail2ban/jail.local' [line 281]:
section 'sshd' already exists
```

This caused fail2ban to crash and the socket to go offline.

**Diagnosis:**
```bash
sudo fail2ban-server --test
```

**Fix:** Located and deleted the duplicate [sshd] block from jail.local,
keeping only the custom block added at the top of the JAILS section.

**Lesson:** Always run `sudo fail2ban-server --test` before restarting
fail2ban after config changes. Silent config errors will crash the service.

---

### Verification

Started and enabled fail2ban:
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

Confirmed jail loaded with correct settings:
```bash
sudo fail2ban-client status sshd
sudo tail -n 20 /var/log/fail2ban.log
```

Log confirmed:
```
maxRetry: 3
findtime: 300
banTime: 600
Added logfile: '/var/log/auth.log'
Jail 'sshd' started
```

---

### Live Ban Test

Triggered ban from Windows host (192.168.56.1) using forced password auth:
```powershell
ssh -p 2222 -o PreferredAuthentications=password -o PubkeyAuthentication=no fakeuser@192.168.56.10
```

Result from fail2ban.log:
```
00:55:32 - Found 192.168.56.1  (attempt 1)
00:55:33 - Found 192.168.56.1  (attempt 2)
00:55:35 - Found 192.168.56.1  (attempt 3)
00:55:36 - NOTICE [sshd] Ban 192.168.56.1
```

Ban fired on exactly the 3rd attempt within 4 seconds. ✓

**Note:** The ban kicked the active SSH session — all connections from a
banned IP are dropped immediately including existing sessions. Recovery
required direct console access to unban and reconnect.

Unbanned:
```bash
sudo fail2ban-client set sshd unbanip 192.168.56.1
```

Final status confirmed:
- Currently banned: 0
- Total banned: 1 (recorded in persistent SQLite database)

---

### Defense Layers After Exp003

| Layer | Tool | What it does |
|---|---|---|
| 1 | SSH hardening (Exp002) | Port 2222, key auth only, no root, no passwords |
| 2 | UFW (Exp001) | Static firewall — only port 2222 open |
| 3 | fail2ban (Exp003) | Dynamic blocking — bans IPs after 3 failed attempts |

---

### Screenshots
- `images/exp003/exp003-01-jail-active.png`
- `images/exp003/exp003-02-ban-fired.png`
- `images/exp003/exp003-03-config-verified.png`
- `images/exp003/exp003-04-clean-ban.png`

### Key Takeaways

- fail2ban adds dynamic threat response on top of static firewall rules
- UFW sets the baseline, fail2ban responds to live behavior
- Always use jail.local not jail.conf for custom config
- Always test config with `fail2ban-server --test` before restarting
- A banned IP loses all access immediately including active sessions
- Persistent SQLite database survives restarts and tracks ban history

---

## Experiment 004 — Splunk SIEM Log Ingestion

**Date:** 2026-05-24
**System:** Ubuntu-Server-01 (192.168.56.10) → Windows Host (192.168.56.1)
**Status:** ✅ Complete

### Objective
Deploy Splunk Enterprise on the Windows host and configure a Universal Forwarder on Ubuntu-Server-01 to ship auth.log and fail2ban.log into Splunk for real-time monitoring, dashboarding, and alerting.

---

### Architecture

```
Ubuntu-Server-01 (192.168.56.10)
  └── Splunk Universal Forwarder
        └── ships logs on port 9997
              └── Windows Host (192.168.56.1)
                    └── Splunk Enterprise (localhost:8000)
```

---

### Components Installed

| Component | Version | Location | Purpose |
|---|---|---|---|
| Splunk Enterprise | 10.4.0 | Windows Host | SIEM — receives, indexes, visualizes logs |
| Splunk Universal Forwarder | 10.4.0 | Ubuntu-Server-01 | Ships log files to Splunk |

---

### Configuration

#### Splunk Enterprise (Windows)
- Installed to: C:\Program Files\Splunk
- Web UI: http://localhost:8000
- Receiving port: 9997
- Windows Firewall rule added: Splunk Receiver 9997 (TCP inbound)
- Credentials: [see Bitwarden — "Splunk Enterprise - Home Lab"]

#### Universal Forwarder (Ubuntu)
- Installed to: /opt/splunkforwarder
- Forward server: 192.168.56.1:9997
- Runs as system user: splunkfwd
- Credentials: [see Bitwarden — "Splunk Forwarder - Ubuntu"]

#### Log Files Monitored
| File | Sourcetype | Contents |
|---|---|---|
| /var/log/auth.log | linux_secure | SSH login attempts, sudo usage, auth events |
| /var/log/fail2ban.log | fail2ban | Ban/unban events, detected intrusion attempts |

---

### Brute Force Simulation

Temporarily enabled password authentication to generate real attack data:

```bash
# Enabled in both config files
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/50-cloud-init.conf
sudo systemctl restart ssh
```

Simulated brute force from Windows host (192.168.56.1):
```powershell
for ($i=1; $i -le 3; $i++) { ssh -p 2222 -o PreferredAuthentications=password fakeuser@192.168.56.10 }
```

Result:
- 3 failed password attempts logged to auth.log
- fail2ban detected attempts and banned 192.168.56.1
- Ban event shipped to Splunk via forwarder

Re-hardened immediately after test:
```bash
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config.d/50-cloud-init.conf
sudo systemctl restart ssh
```

---

### Splunk Queries Used

```
# All incoming events
index=main

# Failed SSH login attempts
index=main sourcetype=linux_secure "Failed password"

# fail2ban events (bans, unbans, detections)
index=main sourcetype=fail2ban

# Failed SSH attempts over time (dashboard query)
index=main sourcetype=linux_secure "Failed password" | timechart count by host
```

---

### Dashboard Built
- **Name:** SSH Attack Monitor
- **Panel:** Failed SSH Attempts Over Time (Column Chart)
- Shows brute force spike at time of simulated attack

---

### Alert Created
- **Name:** fail2ban Ban Detected
- **Type:** Real-time
- **Trigger:** Per-Result — fires on every ban event
- **Severity:** High
- **Action:** Add to Triggered Alerts
- **Query:** `index=main sourcetype=fail2ban "Ban"`

---

### Screenshots
- `images/exp004/exp004-01-splunk-login.png`
- `images/exp004/exp004-02-splunk-receiving-port.png`
- `images/exp004/exp004-03-splunk-firewall-rule.png`
- `images/exp004/exp004-04-splunk-first-events.png`
- `images/exp004/exp004-05-splunk-failed-password-events.png`
- `images/exp004/exp004-06-fail2ban-ban-event.png`
- `images/exp004/exp004-07-splunk-fail2ban-events.png`
- `images/exp004/exp004-08-splunk-dashboard-chart.png`
- `images/exp004/exp004-09-splunk-dashboard-final.png`
- `images/exp004/exp004-10-splunk-alert-setup.png`

### Key Takeaways
- Universal Forwarder is lightweight — runs silently on Ubuntu, no performance impact
- Port 9997 must be open on the receiving Splunk server (Windows Firewall rule required)
- fail2ban's ignoreself rule prevents banning localhost — attacks must come from external IP
- Ubuntu 24.04 has /etc/ssh/sshd_config.d/50-cloud-init.conf which overrides main sshd_config
- Splunk indexes data by sourcetype — labeling correctly at ingest makes searching much easier
- Real-time alerts in Splunk fire per event — useful for high-priority detections like bans

---

### Defense Layers After Exp004

| Layer | Tool | What it does |
|---|---|---|
| 1 | SSH hardening (Exp002) | Port 2222, key auth only, no root |
| 2 | UFW (Exp001) | Static firewall — only port 2222 open |
| 3 | fail2ban (Exp003) | Dynamic blocking — bans IPs after 3 failed attempts |
| 4 | Splunk SIEM (Exp004) | Real-time log ingestion, dashboards, and alerts |


---

## Screenshots Taken

### Setup
- images/setup/setup-01-ubuntu-install-progress.png
- images/setup/setup-02-ubuntu-first-login.png
- images/setup/setup-03-ubuntu-ip-address.png
- images/setup/setup-04-ubuntu-updates-complete.png
- images/setup/setup-05-ubuntu-netplan-config.png
- images/setup/setup-06-ubuntu-static-ip-confirmed.png
- images/setup/setup-07-ubuntu-ssh-connected.png
- images/setup/setup-08-ubuntu-ufw-status.png

### Exp001
- images/exp001/exp001-01-pfsense-firewall-rules.png

### Exp002
- images/exp002/exp002-01-ssh-hardening-verified.png
- images/exp002/exp002-02-ssh-password-disabled.png

### Exp003
- images/exp003/exp003-01-jail-active.png
- images/exp003/exp003-02-ban-fired.png
- images/exp003/exp003-03-config-verified.png
- images/exp003/exp003-04-clean-ban.png

### Exp004
- images/exp004/exp004-01-splunk-login.png
- images/exp004/exp004-02-splunk-receiving-port.png
- images/exp004/exp004-03-splunk-firewall-rule.png
- images/exp004/exp004-04-splunk-first-events.png
- images/exp004/exp004-05-splunk-failed-password-events.png
- images/exp004/exp004-06-fail2ban-ban-event.png
- images/exp004/exp004-07-splunk-fail2ban-events.png
- images/exp004/exp004-08-splunk-dashboard-chart.png
- images/exp004/exp004-09-splunk-dashboard-final.png
- images/exp004/exp004-10-splunk-alert-setup.png
