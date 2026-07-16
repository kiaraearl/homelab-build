# Experiment 002 — SSH Hardening

## Overview
| Field | Details |
|---|---|
| **Experiment** | Exp002 |
| **Title** | SSH Hardening |
| **Date** | 2026-05-23 |
| **Status** | Complete |
| **System** | Ubuntu-Server-01 @ 192.168.56.10 |
| **Cert Connection** | Security+ / CySA+ |

---

## Objective
Harden the SSH configuration on Ubuntu-Server-01 to reduce attack surface, eliminate password-based authentication, and enforce key-based access only.

---

## Why This Matters
SSH is the most commonly targeted remote access service on Linux systems. Default SSH configurations are permissive — they allow password authentication, root login, and run on the well-known port 22 which automated scanners probe constantly. Hardening SSH is one of the most impactful controls you can apply to a Linux server and maps directly to:
- **Security+** — Hardening, attack surface reduction, authentication controls
- **CySA+** — System hardening, credential management, intrusion detection

---

## What I Did

### 1. Modified /etc/ssh/sshd_config

| Setting | Before | After | Reason |
|---|---|---|---|
| Port | 22 | 2222 | Avoid automated bot scans on default port |
| PermitRootLogin | prohibit-password | no | Prevent any root SSH access |
| MaxAuthTries | 6 | 3 | Limit brute force attempts |
| LoginGraceTime | 2m | 30s | Reduce attack window |
| X11Forwarding | yes | no | Server has no GUI — unnecessary exposure |
| PasswordAuthentication | yes | no | Keys only, no password login |

### 2. Updated UFW Firewall Rules
- Opened port 2222/tcp
- Removed OpenSSH rule (port 22)
- Result: Only port 2222 accepts SSH connections

### 3. Generated SSH Key Pair on Windows
- Algorithm: ed25519 (modern, most secure)
- Private key stays on Windows PC only
- Public key copied to server's ~/.ssh/authorized_keys
- Passphrase set on private key for additional protection

### 4. Disabled Password Authentication
- Set `PasswordAuthentication no` in sshd_config
- Set `KbdInteractiveAuthentication no`
- Set `UsePAM no`
- **Key issue discovered:** /etc/ssh/sshd_config.d/50-cloud-init.conf was overriding main config settings
- **Fix:** Updated 50-cloud-init.conf directly to match hardened settings

---

## Verification

```bash
sudo sshd -T | grep -E "port|permitrootlogin|maxauthtries|logingracetime|x11forwarding"
```

- Tested password login with `-o PreferredAuthentications=password` — returned `Permission denied (publickey)`
- Confirmed key-based login works with passphrase

---

## Screenshots
- `images/exp002/exp002-01-ssh-hardening-verified.png` — hardening settings confirmed live
- `images/exp002/exp002-02-ssh-password-disabled.png` — password authentication rejected

---

## Lessons Learned
- Ubuntu 24.04 includes `/etc/ssh/sshd_config.d/` which can override main config — always check this directory
- `UsePAM yes` can bypass `PasswordAuthentication no` — both must be set consistently
- Always verify changes with `sudo sshd -t` before restarting to catch config errors
- Never close your active SSH session before confirming key auth works — you can lock yourself out
- ed25519 is the preferred key algorithm — faster and more secure than RSA for personal use

---

## Cert Connections
| Cert | Domain | Topic |
|---|---|---|
| Security+ | Implementation | SSH hardening, authentication controls |
| CySA+ | Security Operations | System hardening, credential management |

## Related Experiments

- [Exp001 — pfSense Firewall Rules](../Exp001/exp001-pfsense-firewall-rules.md)
- [Exp003 — fail2ban](../Exp003/exp003-fail2ban.md)
- [Exp004 — Splunk SIEM](../Exp004/exp004-splunk-siem.md)
- [Exp005 — Nessus Vulnerability Scan](../Exp005/exp005-nessus-vulnerability-scan.md)
- [Exp006 — Active Directory](../Exp006/exp006-active-directory.md)