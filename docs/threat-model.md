<div align="center">

# ⚠️ Threat Model — Wazuh SIEM Deployment

[![STRIDE](https://img.shields.io/badge/Framework-STRIDE-red?style=for-the-badge)]()
[![MITRE](https://img.shields.io/badge/Mapped%20To-MITRE%20ATT%26CK-blue?style=for-the-badge)]()
[![Microsoft TM](https://img.shields.io/badge/Methodology-Microsoft%20Threat%20Modeling-0078D4?style=for-the-badge&logo=microsoft)]()

</div>

---

## Overview

This document applies the **STRIDE threat modeling framework** to the Wazuh SIEM all-in-one deployment described in this lab. Every component of the system — the Ubuntu Server VM, Wazuh Manager, Indexer, Dashboard, and network access paths — is analyzed for threats across all six STRIDE categories.

STRIDE threats are mapped to **MITRE ATT&CK techniques** where applicable, and mitigations are provided for each identified threat.

---

## System Decomposition

Before modeling threats, we identify the key assets and trust boundaries.

### Assets
| Asset | Description | Criticality |
|---|---|---|
| Wazuh Manager | Core detection and correlation engine | 🔴 Critical |
| Wazuh Indexer | All stored alert and event data | 🔴 Critical |
| Wazuh Dashboard | Web UI — admin credentials, alert visibility | 🔴 Critical |
| Ubuntu Server | Host OS for the entire stack | 🔴 Critical |
| SSH Access (Port 22) | Remote admin terminal | 🟠 High |
| Alert Data | Security events, logs, threat intelligence | 🟠 High |
| Agent Keys | `/var/ossec/etc/client.keys` | 🟠 High |
| Admin Credentials | Wazuh dashboard `admin` account | 🟠 High |
| Self-Signed TLS Cert | HTTPS certificate for dashboard | 🟡 Medium |

### Trust Boundaries
```
[Internet / External]
        │
        │  ← Trust Boundary 1: Network perimeter
        ▼
[Windows Host Machine]
        │
        │  ← Trust Boundary 2: Hypervisor boundary
        ▼
[Ubuntu Server VM — 192.168.1.20]
        │
        ├── SSH (Port 22)      ← Trust Boundary 3: Auth boundary
        │
        ├── Wazuh Dashboard    ← Trust Boundary 4: Web auth boundary
        │       (Port 443)
        │
        └── Wazuh Internal     ← Trust Boundary 5: Inter-process boundary
            (Manager ↔ Indexer ↔ Dashboard)
```

---

## STRIDE Analysis

### 🔵 S — Spoofing

> **Definition:** An attacker impersonates a legitimate user, system, or component.

---

#### THREAT-S-01: Spoofing SSH Identity
| Field | Detail |
|---|---|
| **Target** | SSH service (Port 22) |
| **Description** | An attacker performs a brute-force or credential stuffing attack against the SSH login to impersonate the `abeg` admin user |
| **MITRE ATT&CK** | T1110 — Brute Force |
| **Likelihood** | 🟠 Medium (exposed on local network) |
| **Impact** | 🔴 Critical — full OS access |
| **Mitigation** | Disable password auth; use SSH key pairs only. Implement `fail2ban`. Restrict SSH to specific source IPs via UFW |

---

#### THREAT-S-02: Spoofing Wazuh Agent Identity
| Field | Detail |
|---|---|
| **Target** | Wazuh Manager — Agent Enrollment (Port 1515) |
| **Description** | A rogue machine registers itself as a legitimate Wazuh agent to inject false security events or consume manager resources |
| **MITRE ATT&CK** | T1078 — Valid Accounts |
| **Likelihood** | 🟡 Low (internal network only in this lab) |
| **Impact** | 🟠 High — false alert injection, alert flooding |
| **Mitigation** | Use agent key pre-sharing (`manage_agents`). Enable agent verification with passwords. Restrict port 1515 via firewall to known agent IPs |

---

#### THREAT-S-03: Spoofing Dashboard Admin Identity
| Field | Detail |
|---|---|
| **Target** | Wazuh Dashboard login (Port 443) |
| **Description** | Attacker uses stolen or guessed `admin` credentials to access the dashboard and masquerade as the administrator |
| **MITRE ATT&CK** | T1078.001 — Default Accounts |
| **Likelihood** | 🟠 Medium (default credentials are well-known) |
| **Impact** | 🔴 Critical — full SIEM visibility and config control |
| **Mitigation** | Change default admin password immediately post-install. Enforce strong password policy. Add MFA when available. Restrict dashboard access to management VLAN only |

---

### 🟤 T — Tampering

> **Definition:** Unauthorized modification of data or system components.

---

#### THREAT-T-01: Tampering with Alert Data in Indexer
| Field | Detail |
|---|---|
| **Target** | Wazuh Indexer (OpenSearch) — Port 9200 |
| **Description** | An attacker with network access to port 9200 directly modifies or deletes alert indices, covering tracks of an intrusion |
| **MITRE ATT&CK** | T1565.001 — Stored Data Manipulation |
| **Likelihood** | 🟠 Medium (port 9200 should never be externally exposed) |
| **Impact** | 🔴 Critical — destroys forensic evidence |
| **Mitigation** | Block port 9200 externally via UFW. Enable TLS on OpenSearch. Use role-based access control (RBAC) to restrict index deletion |

---

#### THREAT-T-02: Tampering with Wazuh Rules
| Field | Detail |
|---|---|
| **Target** | `/var/ossec/ruleset/rules/` and `/var/ossec/etc/rules/local_rules.xml` |
| **Description** | An attacker with OS-level access modifies detection rules to blind the SIEM to specific attack patterns |
| **MITRE ATT&CK** | T1562.001 — Disable or Modify Tools |
| **Likelihood** | 🟡 Low (requires prior OS compromise) |
| **Impact** | 🔴 Critical — silently disables threat detection |
| **Mitigation** | Enable Wazuh File Integrity Monitoring (FIM) on `/var/ossec/etc/` and `/var/ossec/ruleset/`. Alert on any rule file change. Restrict write permissions |

---

#### THREAT-T-03: Tampering with ossec.conf
| Field | Detail |
|---|---|
| **Target** | `/var/ossec/etc/ossec.conf` |
| **Description** | Attacker modifies the main config to disable log collection, active response, or agent communication |
| **MITRE ATT&CK** | T1562.001 — Disable or Modify Tools |
| **Likelihood** | 🟡 Low |
| **Impact** | 🔴 Critical |
| **Mitigation** | FIM on config files. Immutable flag: `sudo chattr +i /var/ossec/etc/ossec.conf` in production |

---

### 🟣 R — Repudiation

> **Definition:** An attacker performs malicious actions and denies having done so; or legitimate actions cannot be attributed.

---

#### THREAT-R-01: Admin Action Repudiation on Dashboard
| Field | Detail |
|---|---|
| **Target** | Wazuh Dashboard admin actions |
| **Description** | An administrator (or attacker using admin credentials) makes changes — deleting alerts, modifying settings — with no audit trail |
| **MITRE ATT&CK** | T1070 — Indicator Removal |
| **Likelihood** | 🟠 Medium |
| **Impact** | 🟠 High — no accountability |
| **Mitigation** | Enable Wazuh API audit logging. Forward dashboard audit logs back into the SIEM itself. Use individual named accounts rather than shared `admin` |

---

#### THREAT-R-02: SSH Session Repudiation
| Field | Detail |
|---|---|
| **Target** | SSH access logs |
| **Description** | An attacker clears `/var/log/auth.log` after gaining access, removing evidence of their SSH session |
| **MITRE ATT&CK** | T1070.002 — Clear Linux or Mac System Logs |
| **Likelihood** | 🟠 Medium |
| **Impact** | 🟠 High |
| **Mitigation** | Configure Wazuh to forward `/var/log/auth.log` in real-time to the indexer — once ingested, logs cannot be deleted retroactively. Use `auditd` for enhanced shell command logging |

---

### 🔴 I — Information Disclosure

> **Definition:** Exposure of sensitive information to unauthorized parties.

---

#### THREAT-I-01: Dashboard Credential Exposure (Self-Signed TLS)
| Field | Detail |
|---|---|
| **Target** | Wazuh Dashboard HTTPS (Port 443) |
| **Description** | The self-signed certificate allows a MITM attacker on the local network to intercept dashboard traffic including credentials, since browsers cannot verify the cert's authenticity |
| **MITRE ATT&CK** | T1557 — Adversary-in-the-Middle |
| **Likelihood** | 🟡 Low (lab/local network) |
| **Impact** | 🟠 High — credential theft |
| **Mitigation** | Replace self-signed cert with a CA-signed certificate (Let's Encrypt or internal CA). Pin the certificate in browser settings for lab use |

---

#### THREAT-I-02: Wazuh API Credential Exposure
| Field | Detail |
|---|---|
| **Target** | Wazuh REST API (Port 55000) |
| **Description** | API credentials or JWT tokens are transmitted or stored in plaintext in scripts or shell history |
| **MITRE ATT&CK** | T1552.001 — Credentials In Files |
| **Likelihood** | 🟠 Medium (common in lab setups) |
| **Impact** | 🟠 High — API gives full SIEM control |
| **Mitigation** | Store tokens in environment variables, not scripts. Clear shell history of credential commands. Use short-lived JWT tokens |

---

#### THREAT-I-03: Alert Data Leakage via Indexer API
| Field | Detail |
|---|---|
| **Target** | OpenSearch REST API (Port 9200) |
| **Description** | Port 9200 exposes all indexed alert data without authentication if TLS and auth are not properly configured |
| **MITRE ATT&CK** | T1530 — Data from Cloud Storage |
| **Likelihood** | 🟠 Medium |
| **Impact** | 🔴 Critical — full security event disclosure |
| **Mitigation** | Block port 9200 at firewall. Ensure OpenSearch security plugin is enabled (it is by default in Wazuh 4.x). Never expose 9200 externally |

---

### 🟡 D — Denial of Service

> **Definition:** Making a system or component unavailable to legitimate users.

---

#### THREAT-D-01: Alert Flooding — Agent Event Storm
| Field | Detail |
|---|---|
| **Target** | Wazuh Manager |
| **Description** | A compromised or misconfigured agent sends thousands of events per second, overwhelming the Manager's analysis engine and filling disk with alert logs |
| **MITRE ATT&CK** | T1499 — Endpoint Denial of Service |
| **Likelihood** | 🟠 Medium |
| **Impact** | 🟠 High — SIEM becomes unavailable |
| **Mitigation** | Set per-agent event rate limits in `ossec.conf`. Monitor agent event rates. Implement disk usage alerts |

---

#### THREAT-D-02: Disk Exhaustion via Indexer
| Field | Detail |
|---|---|
| **Target** | Wazuh Indexer — disk storage |
| **Description** | The OpenSearch indexer continuously writes alert data. Without index lifecycle management, the disk fills completely, crashing the entire stack. **This was observed in this lab** (disk at 85.8% post-install) |
| **MITRE ATT&CK** | T1485 — Data Destruction (indirect) |
| **Likelihood** | 🔴 High (already occurring) |
| **Impact** | 🔴 Critical — entire stack crashes |
| **Mitigation** | Configure Index Lifecycle Management (ILM) to auto-delete indices older than N days. Set up disk usage monitoring alerts. Dedicate a separate volume to `/var/lib/wazuh-indexer/` |

---

#### THREAT-D-03: SSH Brute Force Causing Lockout
| Field | Detail |
|---|---|
| **Target** | SSH service |
| **Description** | An attacker floods SSH with login attempts, triggering account lockout and denying legitimate admin access |
| **MITRE ATT&CK** | T1110 — Brute Force |
| **Likelihood** | 🟠 Medium |
| **Impact** | 🟠 High — loss of admin access |
| **Mitigation** | Install `fail2ban`. Use SSH key auth (immune to password brute force). Keep a console backup access method (VirtualBox console) |

---

### ⚫ E — Elevation of Privilege

> **Definition:** An attacker gains access or permissions beyond what they are authorized for.

---

#### THREAT-E-01: Linux Privilege Escalation via Wazuh Processes
| Field | Detail |
|---|---|
| **Target** | Wazuh Manager processes running as root |
| **Description** | Several Wazuh daemons run with elevated privileges. A vulnerability in these processes could allow an attacker who has compromised a low-privilege account to escalate to root |
| **MITRE ATT&CK** | T1068 — Exploitation for Privilege Escalation |
| **Likelihood** | 🟡 Low (requires prior foothold) |
| **Impact** | 🔴 Critical |
| **Mitigation** | Keep Wazuh updated. Apply OS security patches (`sudo apt upgrade`). Use AppArmor profiles. Run Wazuh vulnerability detection on itself |

---

#### THREAT-E-02: Wazuh API Role Escalation
| Field | Detail |
|---|---|
| **Target** | Wazuh REST API RBAC system |
| **Description** | A low-privilege API user exploits a misconfiguration in role mappings to gain admin-level API access |
| **MITRE ATT&CK** | T1078.003 — Local Accounts |
| **Likelihood** | 🟡 Low |
| **Impact** | 🔴 Critical |
| **Mitigation** | Follow principle of least privilege for all API users. Audit role mappings regularly via API. Do not use the `admin` account for integrations |

---

#### THREAT-E-03: Container/VM Escape (Future Risk)
| Field | Detail |
|---|---|
| **Target** | VirtualBox hypervisor |
| **Description** | A future attacker who compromises the VM attempts a hypervisor escape to reach the Windows host machine |
| **MITRE ATT&CK** | T1611 — Escape to Host |
| **Likelihood** | 🟢 Very Low (VirtualBox, lab environment) |
| **Impact** | 🔴 Critical — full host compromise |
| **Mitigation** | Keep VirtualBox updated. Disable unused VM features (shared clipboard, drag-drop). Snapshot the VM regularly |

---

## STRIDE Summary Matrix

| ID | Category | Target | Likelihood | Impact | Status |
|---|---|---|---|---|---|
| S-01 | Spoofing | SSH Login | 🟠 Medium | 🔴 Critical | ⚠️ Mitigate |
| S-02 | Spoofing | Agent Enrollment | 🟡 Low | 🟠 High | ⚠️ Mitigate |
| S-03 | Spoofing | Dashboard Login | 🟠 Medium | 🔴 Critical | ⚠️ Mitigate |
| T-01 | Tampering | Indexer Data | 🟠 Medium | 🔴 Critical | ⚠️ Mitigate |
| T-02 | Tampering | Detection Rules | 🟡 Low | 🔴 Critical | ⚠️ Mitigate |
| T-03 | Tampering | ossec.conf | 🟡 Low | 🔴 Critical | ⚠️ Mitigate |
| R-01 | Repudiation | Dashboard Actions | 🟠 Medium | 🟠 High | ⚠️ Mitigate |
| R-02 | Repudiation | SSH Sessions | 🟠 Medium | 🟠 High | ⚠️ Mitigate |
| I-01 | Info Disclosure | TLS Certificate | 🟡 Low | 🟠 High | ⚠️ Mitigate |
| I-02 | Info Disclosure | API Credentials | 🟠 Medium | 🟠 High | ⚠️ Mitigate |
| I-03 | Info Disclosure | Indexer Port 9200 | 🟠 Medium | 🔴 Critical | ⚠️ Mitigate |
| D-01 | Denial of Service | Manager Flooding | 🟠 Medium | 🟠 High | ⚠️ Mitigate |
| D-02 | Denial of Service | Disk Exhaustion | 🔴 High | 🔴 Critical | 🚨 Active Risk |
| D-03 | Denial of Service | SSH Brute Force | 🟠 Medium | 🟠 High | ⚠️ Mitigate |
| E-01 | Privilege Escalation | Wazuh Processes | 🟡 Low | 🔴 Critical | ⚠️ Mitigate |
| E-02 | Privilege Escalation | API RBAC | 🟡 Low | 🔴 Critical | ⚠️ Mitigate |
| E-03 | Privilege Escalation | VM Escape | 🟢 Very Low | 🔴 Critical | 📋 Monitor |

---

## Priority Mitigations (Top 5)

Based on the matrix above, these are the highest-priority actions for hardening this deployment:

| Priority | Action | Addresses |
|---|---|---|
| 1 | Configure Index Lifecycle Management (ILM) | D-02 — Active disk exhaustion risk |
| 2 | Disable SSH password auth; use key pairs + fail2ban | S-01, D-03 |
| 3 | Change default Wazuh admin password; restrict dashboard access | S-03 |
| 4 | Block ports 9200 and 55000 externally via UFW | I-03, I-02 |
| 5 | Enable FIM on Wazuh config and rule directories | T-01, T-02, T-03 |

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 204  
*Threat model prepared using the Microsoft STRIDE framework*  
[![GitHub](https://img.shields.io/badge/GitHub-ICE1945-181717?style=flat-square&logo=github)](https://github.com/ICE1945)

</div>

