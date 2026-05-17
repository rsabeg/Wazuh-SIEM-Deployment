# 📅 Changelog — Wazuh SIEM Deployment

All notable changes and milestones of this deployment are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.2.0] — 2026-05-18

### Added
- `docs/architecture.md` — Full system architecture documentation with component breakdown, data flow diagram, and resource utilization table
- `docs/threat-model.md` — Complete STRIDE threat model covering 17 identified threats across all 6 STRIDE categories, mapped to MITRE ATT&CK
- `docs/troubleshooting.md` — Real-world troubleshooting guide covering 10 common issues including the disk exhaustion problem observed in this lab
- `config/ossec.conf` — Annotated sample Wazuh Manager configuration with lab defaults and recommended hardening settings
- `config/local_rules.xml` — Custom detection rules covering SSH brute force, privilege escalation, FIM on critical files, disk alerts, and agent monitoring
- `COMMANDS.md` — Comprehensive command reference with 13 sections covering all lab commands, alternatives, and advanced future commands
- `CHANGELOG.md` — This file

### Changed
- `README.md` — Upgraded from plain lab report format to professional layout with badges, architecture diagram, results table, and GitHub/LinkedIn footer

---

## [1.1.0] — 2026-05-16

### Added
- `screenshots/` folder with 26 screenshots documenting every step of the deployment
- Full Ubuntu Server 22.04.5 LTS installation walkthrough (screenshots 1–12)
- Post-installation SSH access via PuTTY (screenshots 13, 22, 23)
- Wazuh 4.12.0 installation process (screenshots 14–17)
- Network verification and IP configuration (screenshot 18)
- Dashboard access including TLS bypass and login (screenshots 19–20)
- Wazuh Dashboard overview after successful login (screenshot 21)
- All three service status verifications — Manager, Indexer, Dashboard (screenshots 24–26)

---

## [1.0.0] — 2026-05-16

### Added
- Initial repository created: `Wazuh-SIEM-Deployment`
- `README.md` — Initial lab report document

### Deployment Milestones
- ✅ Oracle VirtualBox VM provisioned (5783 MB RAM, 2 vCPUs, 25 GB VDI disk)
- ✅ Ubuntu Server 22.04.5 LTS installed and configured
- ✅ SSH access established via PuTTY on port 22
- ✅ Java JDK installed as Wazuh prerequisite
- ✅ Wazuh 4.12.0 all-in-one installation completed
- ✅ Wazuh Manager — Active and running
- ✅ Wazuh Indexer (OpenSearch) — Active and running
- ✅ Wazuh Dashboard — Active and running
- ✅ Web dashboard accessible at `https://192.168.1.20`

---

## 🗺️ Roadmap — Planned Future Updates

| Version | Planned Addition | Status |
|---|---|---|
| 1.3.0 | Deploy first Wazuh agent on a Windows endpoint | 📋 Planned |
| 1.3.0 | Enable File Integrity Monitoring on agent | 📋 Planned |
| 1.4.0 | Configure active response (auto IP block on brute force) | 📋 Planned |
| 1.4.0 | Set up Index Lifecycle Management (ILM) to fix disk issue | 📋 Planned |
| 1.5.0 | Integrate with NetGuard NIDS as an alert source | 📋 Planned |
| 1.5.0 | Custom Wazuh rules for NetGuard C++ sensor alerts | 📋 Planned |
| 2.0.0 | Wazuh API integration with NetGuard dashboard | 📋 Planned |
| 2.0.0 | MITRE ATT&CK coverage report | 📋 Planned |

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 203  
[![GitHub](https://img.shields.io/badge/GitHub-ICE1945-181717?style=flat-square&logo=github)](https://github.com/ICE1945)

</div>

