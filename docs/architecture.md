<div align="center">

# 🏗️ System Architecture — Wazuh SIEM Deployment

</div>

---

## Overview

This document describes the complete system architecture of the Wazuh SIEM all-in-one deployment built for this lab. The stack runs entirely inside a single Ubuntu Server 22.04.5 LTS virtual machine hosted on Oracle VirtualBox, with the Windows host machine acting as the management workstation.

---

## High-Level Architecture

```
╔══════════════════════════════════════════════════════════════════╗
║                     WINDOWS HOST MACHINE                         ║
║                                                                  ║
║   ┌─────────────┐          ┌──────────────────────────────────┐ ║
║   │   PuTTY     │──SSH:22──►                                  │ ║
║   │  (SSH Client│          │    Ubuntu Server 22.04.5 LTS     │ ║
║   └─────────────┘          │       IP: 192.168.1.20           │ ║
║                            │                                  │ ║
║   ┌─────────────┐          │  ┌────────────────────────────┐  │ ║
║   │   Firefox   │─HTTPS:443►  │       WAZUH STACK          │  │ ║
║   │  (Dashboard)│          │  │                            │  │ ║
║   └─────────────┘          │  │  ┌──────────────────────┐  │  │ ║
║                            │  │  │   Wazuh Manager      │  │  │ ║
║                            │  │  │   /var/ossec/        │  │  │ ║
║                            │  │  │   Port: 1514, 1515   │  │  │ ║
║                            │  │  │   API: 55000         │  │  │ ║
║                            │  │  └──────────────────────┘  │  │ ║
║                            │  │                            │  │ ║
║                            │  │  ┌──────────────────────┐  │  │ ║
║                            │  │  │   Wazuh Indexer      │  │  │ ║
║                            │  │  │   (OpenSearch)       │  │  │ ║
║                            │  │  │   Port: 9200, 9300   │  │  │ ║
║                            │  │  └──────────────────────┘  │  │ ║
║                            │  │                            │  │ ║
║                            │  │  ┌──────────────────────┐  │  │ ║
║                            │  │  │   Wazuh Dashboard    │  │  │ ║
║                            │  │  │   (OpenSearch Dash)  │  │  │ ║
║                            │  │  │   Port: 443          │  │  │ ║
║                            │  │  └──────────────────────┘  │  │ ║
║                            │  └────────────────────────────┘  │ ║
║                            └──────────────────────────────────┘ ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Component Breakdown

### 🖥️ Host Machine (Windows)
The physical machine running the hypervisor. Acts as the management workstation — all SSH access and dashboard browsing originates here.

| Property | Value |
|---|---|
| Role | Hypervisor Host / Management Workstation |
| Hypervisor | Oracle VirtualBox |
| SSH Client | PuTTY 0.83 |
| Browser | Mozilla Firefox |

---

### 🐧 Virtual Machine — Ubuntu Server 22.04.5 LTS

The guest OS hosting the entire Wazuh stack. Configured as a headless server with no GUI — all interaction is via SSH or the web dashboard.

| Property | Value |
|---|---|
| OS | Ubuntu Server 22.04.5 LTS |
| Kernel | Linux 5.15.0-179-generic x86_64 |
| RAM Allocated | 5783 MB |
| CPU Allocated | 2 vCPUs |
| Disk | 25 GB VDI (VirtualBox Disk Image) |
| Filesystem | ext4 mounted at `/` — 22.47 GB usable |
| Network Interface | enp0s3 (Intel 82540EM Gigabit) |
| IP Address | 192.168.1.20 (DHCPv4) |
| Network Mode | NAT / Host-Only Adapter |

---

### 🛡️ Wazuh Manager

The core component of the Wazuh stack. Responsible for receiving and analyzing security events from agents, running the correlation engine, and triggering active responses.

| Property | Value |
|---|---|
| Version | Wazuh 4.12.0 |
| Process | `wazuh-manager.service` |
| Install Path | `/var/ossec/` |
| Config File | `/var/ossec/etc/ossec.conf` |
| Alerts Log | `/var/ossec/logs/alerts/alerts.log` |
| Agent Port | 1514 (UDP/TCP) |
| Enrollment Port | 1515 (TCP) |
| REST API Port | 55000 (TCP) |
| Memory Usage | ~1.7 GB |
| Status | ✅ Active (running) |

**Key sub-processes running under the Manager:**
- `wazuh-analysisd` — core analysis and correlation engine
- `wazuh-remoted` — agent communication handler
- `wazuh-logcollector` — log collection from local files
- `wazuh-syscheckd` — file integrity monitoring daemon
- `wazuh-monitord` — agent monitoring and keepalive
- `wazuh-execd` — active response executor
- `wazuh-authd` — agent authentication daemon
- `wazuh-db` — agent database manager
- `wazuh-modulesd` — extended modules (vulnerability detection, etc.)

---

### 🔍 Wazuh Indexer (OpenSearch)

The data storage and search backend. Stores all security events, alerts, and agent data in indexed form for fast querying and visualization.

| Property | Value |
|---|---|
| Based On | OpenSearch (Apache-licensed fork of Elasticsearch) |
| Process | `wazuh-indexer.service` |
| Main PID | Java process |
| REST API Port | 9200 (HTTPS) |
| Cluster Port | 9300 (TCP) |
| Data Directory | `/var/lib/wazuh-indexer/` |
| Config File | `/etc/wazuh-indexer/opensearch.yml` |
| Memory Usage | ~1.1 GB |
| Status | ✅ Active (running) |

> ⚠️ **Note:** The indexer is the most resource-intensive component and the primary disk consumer. Monitor `/var/lib/wazuh-indexer/` regularly and set up index rotation to prevent disk exhaustion.

---

### 📊 Wazuh Dashboard (OpenSearch Dashboards)

The web-based visualization and management interface. Provides real-time alert dashboards, MITRE ATT&CK mapping, compliance views (PCI DSS, HIPAA, NIST 800-53), and agent management.

| Property | Value |
|---|---|
| Based On | OpenSearch Dashboards |
| Process | `wazuh-dashboard.service` |
| Main PID | Node.js process |
| Access Port | 443 (HTTPS) |
| Access URL | `https://192.168.1.20` |
| Config File | `/etc/wazuh-dashboard/opensearch_dashboards.yml` |
| Memory Usage | ~206 MB |
| Default User | `admin` |
| TLS | Self-signed certificate (lab environment) |
| Status | ✅ Active (running) |

---

## Data Flow

```
  [External Agents]                    [Wazuh Manager]
  (future endpoints)                   /var/ossec/
        │                                    │
        │── TCP/UDP 1514 ──────────────────► │ wazuh-remoted
        │                                    │     │
        │                                    │     ▼
        │                               wazuh-analysisd
        │                               (rule matching,
        │                                correlation,
        │                                MITRE mapping)
        │                                    │
        │                                    ▼
        │                              alerts.log / alerts.json
        │                                    │
        │                                    ▼
        │                          [Wazuh Indexer :9200]
        │                           (OpenSearch)
        │                           Stores & indexes
        │                           all alert data
        │                                    │
        │                                    ▼
        │                          [Wazuh Dashboard :443]
        │                           (OpenSearch Dashboards)
        │                           Visualizes alerts,
        └──────────────────────────  manages agents,
                                     compliance views
```

---

## Network Port Reference

| Port | Protocol | Component | Purpose |
|---|---|---|---|
| 22 | TCP | Ubuntu SSH | Remote terminal access via PuTTY |
| 443 | TCP | Wazuh Dashboard | HTTPS web interface |
| 1514 | TCP/UDP | Wazuh Manager | Agent event forwarding |
| 1515 | TCP | Wazuh Manager | Agent auto-enrollment |
| 1516 | TCP | Wazuh Manager | Wazuh cluster communication |
| 9200 | TCP | Wazuh Indexer | OpenSearch REST API |
| 9300 | TCP | Wazuh Indexer | OpenSearch cluster transport |
| 55000 | TCP | Wazuh Manager | REST API for external integrations |

---

## Resource Utilization (Observed)

| Resource | Usage | Notes |
|---|---|---|
| RAM | ~50% of 5.7 GB | Indexer + Manager are heaviest consumers |
| CPU | 0.1 – 0.56 load | Spikes during installation and scan |
| Disk (/) | 85.8% of 22.47 GB | Post-install — indexer fills this fast |
| Swap | 8% | Engaged due to Java heap (Indexer) |

> ⚠️ **Disk at 85.8% after initial install is a concern.** In a production environment, a dedicated disk or volume for `/var/lib/wazuh-indexer/` with automated index lifecycle management (ILM) would be mandatory.

---

## Deployment Method

This lab uses Wazuh's **All-in-One** deployment script, which installs and configures all three components (Manager, Indexer, Dashboard) on a single node. This is suitable for lab environments and small-scale monitoring.

```
Production alternatives:
├── Distributed deployment  — Manager, Indexer, Dashboard on separate nodes
├── Cluster deployment      — Multiple Indexer nodes for high availability
└── Cloud deployment        — Wazuh on AWS/Azure/GCP with elastic scaling
```

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 203  
[![GitHub](https://img.shields.io/badge/GitHub-ICE1945-181717?style=flat-square&logo=github)](https://github.com/ICE1945)

</div>

