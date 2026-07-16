# Experiment 006 — Active Directory Domain Services

## Overview
| Field | Details |
|---|---|
| **Experiment** | Exp006 |
| **Title** | Active Directory Domain Services |
| **Date** | 2026-05-25 |
| **Status** | Complete |
| **System** | WinServer2022-DC01 @ 192.168.56.20 |
| **Cert Connection** | Security+ / CySA+ / Network+ |

---

## Objective
Deploy a Windows Server 2022 VM, install and configure Active Directory Domain Services (AD DS), promote it to a Domain Controller, create Organizational Units and domain users, and integrate Windows Security Event logs into the Splunk SIEM from Exp004.

---

## Why This Matters
Active Directory is the backbone of identity and access management in virtually every enterprise environment. SOC analysts interact with AD daily — monitoring authentication events, investigating account changes, and detecting lateral movement. This experiment maps directly to:
- **Security+** — Identity and access management, authentication controls, account management
- **CySA+** — AD event monitoring, SIEM integration, logon analysis, privilege monitoring
- **Network+** — DNS, domain architecture, name resolution

---

## What I Did

### 1. Provisioned Windows Server 2022 VM
- Downloaded Windows Server 2022 Standard Evaluation ISO from Microsoft Evaluation Center (free 180-day trial)
- Created VirtualBox VM named `WinServer2022-DC01` — 4GB RAM, 2 CPUs, 50GB VDI disk
- Configured Adapter 1 as NAT (internet) and Adapter 2 as Host-Only (192.168.56.x network)
- Installed Windows Server 2022 Standard Evaluation with Desktop Experience
- Set Administrator password and saved to Bitwarden

### 2. Configured Static IP and Hostname
Set static IP on Ethernet 2 (Host-Only adapter):

| Setting | Value |
|---|---|
| IP Address | 192.168.56.20 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | (blank) |
| Preferred DNS | 127.0.0.1 |

DNS set to loopback so DC resolves its own domain records after AD promotion. Renamed computer to `DC01` via PowerShell:

```powershell
Rename-Computer -NewName "DC01" -Restart
```

### 3. Installed Active Directory Domain Services
Installed AD DS role via Server Manager → Add Roles and Features → Active Directory Domain Services. Components installed:
- Active Directory Domain Services
- Group Policy Management
- Active Directory module for Windows PowerShell
- Active Directory Administrative Center
- AD DS Snap-Ins and Command-Line Tools

### 4. Promoted DC01 to Domain Controller
Used AD DS Configuration Wizard to create a new forest:

| Setting | Value |
|---|---|
| Deployment | Add a new forest |
| Root domain name | lab.local |
| Forest functional level | Windows Server 2016 |
| DNS server | Enabled |
| Global Catalog | Enabled |
| NetBIOS name | LAB |
| DSRM password | [see Bitwarden — "WinServer2022-DC01 - DSRM Password"] |

After promotion, server rebooted and login screen updated to `LAB\Administrator` — confirming domain is live.

### 5. Created Organizational Units
```powershell
New-ADOrganizationalUnit -Name "SOC_Team" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "IT_Admin" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=lab,DC=local"
```

### 6. Created Domain Users
Created via Active Directory Users and Computers (ADUC) GUI:

| Name | Username | UPN | OU |
|---|---|---|---|
| Kiara Earl | kearl | kearl@lab.local | SOC_Team |
| Jane Doe | jdoe | jdoe@lab.local | SOC_Team |
| John Smith | jsmith | jsmith@lab.local | IT_Admin |

### 7. Integrated with Splunk SIEM
Installed Splunk Universal Forwarder on DC01 and configured it to forward Windows Security Event logs to Splunk on the host:

- Receiver: `192.168.56.1:9997`
- Created `inputs.conf` at `C:\Program Files\SplunkUniversalForwarder\etc\system\local\`:

```ini
[WinEventLog://Security]
index = main
sourcetype = WinEventLog:Security
disabled = false
renderXml = false
```

Started SplunkForwarder service and verified 304 Windows Security Events flowing into Splunk from `host = DC01`.

---

## Verification

```powershell
# Confirm domain is live
Get-ADDomain
```
Confirmed: DNSRoot = `lab.local`, PDCEmulator = `DC01.lab.local`, NetBIOSName = `LAB`

```powershell
# Confirm OUs exist
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName
```

```powershell
# Confirm users exist and are enabled
Get-ADUser -Filter * -Properties DisplayName,Department | Select-Object Name,SamAccountName,UserPrincipalName,Enabled | Format-Table -AutoSize
```

Splunk search confirming log ingestion:
```
index=main sourcetype="WinEventLog:Security"
```
Result: 304 events, host = DC01

---

## Key Event IDs Observed in Splunk

| Event ID | Description | SOC Relevance |
|---|---|---|
| 4624 | Successful Logon | Baseline auth — monitor for unusual hours or sources |
| 4634 | Logoff | Session tracking |
| 4672 | Special Privileges Assigned | Admin logon — high value for privilege monitoring |
| 4720 | User Account Created | Identity lifecycle — should match change requests |
| 5024 | Windows Firewall Started | Service monitoring |

---

## Screenshots
- `images/setup/exp006-01-ws2022-iso-download.png`
- `images/setup/exp006-02-vbox-vm-name-iso.png`
- `images/setup/exp006-02-vbox-vm-hardware-disk.png`
- `images/setup/exp006-03-vbox-network-hostonly.png`
- `images/setup/exp006-04-ws2022-setup-start.png`
- `images/setup/exp006-05-ws2022-edition-select.png`
- `images/setup/exp006-06-ws2022-installing.png`
- `images/setup/exp006-07-ws2022-admin-password.png`
- `images/setup/exp006-08-ws2022-server-manager.png`
- `images/setup/exp006-09-ws2022-desktop-clean.png`
- `images/config/exp006-10-ws2022-static-ip.png`
- `images/config/exp006-11-ws2022-ipconfig-verified.png`
- `images/config/exp006-12-ws2022-renamed-dc01.png`
- `images/config/exp006-13-adds-role-selected.png`
- `images/config/exp006-13-adds-confirmation.png`
- `images/config/exp006-14-adds-install-complete.png`
- `images/config/exp006-15-adds-new-forest.png`
- `images/config/exp006-16-adds-dc-options.png`
- `images/config/exp006-17-adds-prerequisites-passed.png`
- `images/verification/exp006-18-adds-domain-login-screen.png`
- `images/verification/exp006-19-adds-domain-desktop.png`
- `images/verification/exp006-20-adds-get-addomain.png`
- `images/verification/exp006-21-adds-ous-created.png`
- `images/verification/exp006-22-aduc-open.png`
- `images/verification/exp006-23-aduc-ou-structure.png`
- `images/verification/exp006-24-aduc-users-created.png`
- `images/verification/exp006-25-aduc-users-verified.png`
- `images/verification/exp006-26-event-viewer-security-log.png`
- `images/verification/exp006-27-uf-indexer-config.png`
- `images/verification/exp006-28-splunk-ad-logs.png`

---

## Lessons Learned
- AD DS requires a static IP before promotion — if DHCP is active, DNS resolution will break after reboot
- DNS must point to `127.0.0.1` (loopback) on the DC itself — the DC is its own DNS server after promotion
- The DNS delegation warning during promotion is expected in a lab with no parent zone — safe to ignore
- `splunk add monitor WinEventLog://Security` CLI syntax is not supported on Windows Universal Forwarder — use `inputs.conf` directly instead
- Always use `jail.local` not `jail.conf` — this applies to configs generally, always use local override files
- A Domain Controller login screen showing `LAB\Administrator` is the visual proof that AD promotion succeeded
- 304 security events were generated within minutes of the forwarder starting — AD is extremely log-verbose, which is exactly what you want in a monitored environment

---

## Defense Layers After Exp006

| Layer | Tool | What It Does |
|---|---|---|
| 1 | pfSense Firewall (Exp001) | Default deny, specific allow rules |
| 2 | SSH Hardening (Exp002) | Port 2222, key auth only, no root, no passwords |
| 3 | fail2ban (Exp003) | Dynamic blocking — bans IPs after 3 failed attempts |
| 4 | Splunk SIEM (Exp004) | Real-time log ingestion, dashboards, and alerts |
| 5 | Nessus (Exp005) | Vulnerability scanning — validated 0 critical/high findings |
| 6 | Active Directory (Exp006) | Identity & access management, AD event monitoring in Splunk |

---

## Cert Connections
| Cert | Domain | Topic |
|---|---|---|
| Security+ | Identity & Access Management | Active Directory, authentication, account management |
| CySA+ | Security Operations | AD event monitoring, SIEM integration, logon analysis |
| Network+ | Network Services | DNS, domain architecture, name resolution |

## Related Experiments

- [Exp001 — pfSense Firewall Rules](exp001-pfsense-firewall-rules.md)
- [Exp002 — SSH Hardening](exp002-ssh-hardening.md)
- [Exp003 — fail2ban](exp003-fail2ban.md)
- [Exp004 — Splunk SIEM](exp004-splunk-siem.md)
- [Exp005 — Nessus Vulnerability Scan](exp005-nessus-vulnerability-scan.md)
