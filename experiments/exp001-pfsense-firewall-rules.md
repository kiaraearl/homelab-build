# Experiment 001 — pfSense Firewall Rules

## Overview
| Field | Details |
|---|---|
| **Experiment** | Exp001 |
| **Title** | pfSense Firewall Rules |
| **Date** | 2026-05-23 |
| **Status** | ✅ Complete |
| **System** | pfSense-Firewall @ 192.168.56.2 |
| **Cert Connection** | Network+ / Security+ |

---

## Objective
Replace the default allow-all firewall rules on pfSense with a minimal, explicit ruleset following the principle of least privilege. Establish a default deny posture where only required traffic is permitted.

---

## Why This Matters
Firewalls are the first line of defense in any network. The default pfSense ruleset allows all outbound traffic — acceptable for a home router but dangerous in a security lab or enterprise environment. Configuring explicit allow rules with a default deny baseline is a foundational security control that maps directly to:
- **Network+** — Firewall configuration, rule ordering, network segmentation
- **Security+** — Defense in depth, least privilege, attack surface reduction

---

## What I Did

### Default Rules Found
| Rule | Status | Action |
|---|---|---|
| Anti-Lockout Rule | Kept | Protects web UI access — never disable |
| Default allow LAN to any (IPv4) | Disabled | Too permissive |
| Default allow LAN IPv6 to any | Disabled | Too permissive |

### Rules Created (in order)
| Rule | Protocol | Source | Destination | Port | Action |
|---|---|---|---|---|---|
| Allow DNS | UDP | LAN subnets | 192.168.56.2 | 53 | Pass |
| Allow SSH to Ubuntu | TCP | LAN subnets | 192.168.56.10 | 2222 | Pass |
| Allow ICMP ping | ICMP | LAN subnets | Any | Any | Pass |
| Block all other traffic | Any | Any | Any | Any | Block |

### Key Concepts Applied
- **Default deny** — block everything not explicitly allowed
- **Least privilege** — only open what's needed
- **Rule order matters** — pfSense reads top to bottom, first match wins
- Specific allow rules go at top, catch-all block goes at bottom

---

## Verification
- Pinged pfSense (192.168.56.2) from Ubuntu — 128/128 packets received, 0% packet loss ✅
- SSH to Ubuntu on port 2222 still works through firewall ✅

---

## Screenshots
- `images/exp001/exp001-01-pfsense-firewall-rules.png` — final ruleset with default rules disabled

---

## Lessons Learned
- pfSense rule order is critical — first match wins, so specific allows must come before the catch-all block
- The Anti-Lockout Rule must never be removed — it prevents being locked out of the web UI
- Default allow rules ship with pfSense for convenience, not security — always replace them in a lab or production environment
- Testing connectivity after each rule change confirms the ruleset works as intended

---

## Cert Connections
| Cert | Domain | Topic |
|---|---|---|
| Network+ | Network Security | Firewall configuration, rule ordering |
| Security+ | Architecture & Design | Least privilege, default deny, defense in depth |

## Related Experiments

- [Exp002 — SSH Hardening](exp002-ssh-hardening.md)
- [Exp003 — fail2ban](exp003-fail2ban.md)
- [Exp004 — Splunk SIEM](exp004-splunk-siem.md)
- [Exp005 — Nessus Vulnerability Scan](exp005-nessus-vulnerability-scan.md)
- [Exp006 — Active Directory](exp006-active-directory.md)