<div align="center">

# Wazuh SIEM Deployment on Ubuntu Server 22.04

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04.5_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Wazuh](https://img.shields.io/badge/Wazuh-4.12.0-005571?style=for-the-badge&logo=wazuh&logoColor=white)](https://wazuh.com/)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)](https://www.virtualbox.org/)
[![Status](https://img.shields.io/badge/Status-Active%20%26%20Running-brightgreen?style=for-the-badge)]()

<br/>

**Full deployment of the Wazuh open-source SIEM/XDR platform on a virtualized Ubuntu Server environment — covering VM provisioning, OS installation, SSH access via PuTTY, and Wazuh all-in-one stack setup with a live web dashboard.**

<br/>

| Field | Details |
|---|---|
| **Course** | Threat Modelling and Security Monitoring Sessional |
| **Course Code** | SEC 204 |
| **Student** | Ragib Shahriar Abeg |
| **ID** | 2304017 |
| **Submitted To** | Masud Rana (Lecturer) |

</div>

---

## Objective

Deploy and configure **Wazuh 4.12.0** — an open-source Security Information and Event Management (SIEM) and Extended Detection & Response (XDR) platform — on **Ubuntu Server 22.04.5 LTS** running inside Oracle VirtualBox. Access the server remotely via PuTTY over SSH and verify the full stack through the Wazuh web dashboard.

---

## 🧰 Stack & Tools

| Category | Tool / Version |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu Server 22.04.5 LTS |
| SIEM Platform | Wazuh 4.12.0 (All-in-One) |
| SSH Client | PuTTY 0.83 |
| Browser | Mozilla Firefox |
| Network | NAT / Host-Only — IP `192.168.1.20` |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│              Windows Host Machine                │
│                                                  │
│  ┌─────────────────────────────────────────┐    │
│  │        VirtualBox VM                    │    │
│  │   Ubuntu Server 22.04.5 LTS            │    │
│  │   IP: 192.168.1.20                     │    │
│  │                                         │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────┐ │    │
│  │  │  Wazuh   │  │  Wazuh   │  │Wazuh │ │    │
│  │  │ Manager  │  │ Indexer  │  │ Dash │ │    │
│  │  │(OSSEC)   │  │(OpenSrch)│  │board │ │    │
│  │  └──────────┘  └──────────┘  └──────┘ │    │
│  └─────────────────────────────────────────┘    │
│                                                  │
│   PuTTY (SSH :22) ──────────► VM Terminal       │
│   Firefox (HTTPS :443) ──────► Wazuh Dashboard  │
└─────────────────────────────────────────────────┘
```

---

## Lab Walkthrough

### 1 — Download Ubuntu Server ISO

Downloaded Ubuntu Server 22.04.5 LTS from the official Canonical website.

![Ubuntu ISO Download](screenshots/1.png)

---

### 2 — Create Virtual Machine in VirtualBox

Configured the VM with name, ISO image path, OS type (Linux / Ubuntu 64-bit), RAM, CPU, and disk size.

![VM Name & OS](screenshots/2.png)
![Hardware — RAM & CPU](screenshots/3.png)
![Hard Disk — 25 GB VDI](screenshots/4.png)

---

### 3 — Ubuntu Server Installation

Walked through the full Ubuntu Server text-based installer — language, install type, network, proxy, mirror, and storage layout.

![GRUB Boot Menu](screenshots/5.png)
![Language Selection — English](screenshots/6.png)
![Installation Type — Ubuntu Server](screenshots/7.png)
![Network Configuration — DHCPv4](screenshots/8.png)
![Proxy Configuration — None](screenshots/9.png)
![Archive Mirror — archive.ubuntu.com](screenshots/10.png)
![Guided Storage Layout](screenshots/11.png)
![Storage Summary — ext4 on /](screenshots/12.png)

---

### 4 — Post-Installation Login & SSH Access

Logged into the server directly from the VirtualBox console, then connected remotely using PuTTY over SSH.

![First Login — Ubuntu Console](screenshots/13.png)
![PuTTY — Host Configuration (192.168.1.20 :22)](screenshots/22.png)
![PuTTY — Successful SSH Session](screenshots/23.png)

---

### 5 — Wazuh Installation

Installed Java (required dependency), downloaded the official Wazuh install script, and ran the all-in-one deployment which provisions the Manager, Indexer, and Dashboard in a single pass.

![Installing Java JDK](screenshots/14.png)
![Downloading wazuh-install.sh](screenshots/15.png)
![Wazuh Installation in Progress](screenshots/16.png)
![Installation Complete — Credentials Displayed](screenshots/17.png)

---

### 6 — Network Verification

Confirmed the VM's IP address (`192.168.1.20`) using `ip a` to ensure it is reachable from the host browser.

![IP Address Verification](screenshots/18.png)

---

### 7 — Wazuh Web Dashboard

Navigated to `https://192.168.1.20` from the host browser. Bypassed the self-signed TLS certificate warning (expected in a lab environment) and logged in with the auto-generated admin credentials.

![Browser TLS Warning — Self-Signed Cert](screenshots/19.png)
![Wazuh Login Page](screenshots/20.png)
![Wazuh Dashboard Overview](screenshots/21.png)

---

### 8 — Service Status Verification

Confirmed all three Wazuh components are active and running via `systemctl`.

![Wazuh Manager — Active (running)](screenshots/24.png)
![Wazuh Indexer — Active (running)](screenshots/25.png)
![Wazuh Dashboard — Active (running)](screenshots/26.png)

---

## Results

| Component | Status |
|---|---|
| Ubuntu Server 22.04.5 LTS | ✅ Installed & Running |
| SSH Access via PuTTY | ✅ Connected on port 22 |
| Wazuh Manager | ✅ Active (running) |
| Wazuh Indexer (OpenSearch) | ✅ Active (running) |
| Wazuh Dashboard | ✅ Active (running) |
| Web Dashboard Access | ✅ Accessible at https://192.168.1.20 |

---

## Key Commands Used

```bash
# Check IP address
ip a
hostname -I

# Update system
sudo apt update && sudo apt upgrade -y

# Install Java (Wazuh dependency)
sudo apt install default-jdk -y

# Download Wazuh installer
sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh

# Run all-in-one installation
sudo bash wazuh-install.sh -a

# Verify services
systemctl status wazuh-manager --no-pager
systemctl status wazuh-indexer --no-pager
systemctl status wazuh-dashboard --no-pager
```

---

## Conclusion

This lab successfully demonstrated the end-to-end deployment of a **Wazuh SIEM/XDR stack** on a virtualized Ubuntu Server environment. All three core components — Manager, Indexer, and Dashboard — were verified as active and operational. The web dashboard is fully accessible and ready for endpoint agent enrollment, log analysis, threat hunting, and compliance monitoring.

The deployment provides a solid foundation for further NetGuard integration, where Wazuh can serve as the central alert aggregation and correlation layer.

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 204

[![GitHub](https://img.shields.io/badge/GitHub-rsabeg-181717?style=flat-square&logo=github)](https://github.com/rsabeg)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-rsabeg-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/rsabeg)

</div>

