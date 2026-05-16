# Wazuh SIEM Installation on Ubuntu Server 22.04

### Lab Work
**Student Name:** Ragib Shahriar Abeg  
**Student ID:** 2304017
**Course Name:** Threat Modelling and Security Monitoring Sessional  
**Course Code:** SEC 203
**Submitted to:** Masud Rana (Lecturer)

---

## Objective
To install and configure **Wazuh** (Open Source Security Monitoring Platform) on **Ubuntu Server 22.04 LTS** using VirtualBox and access it via PuTTY and Web Dashboard.

---

## Tools & Technologies Used

- **Virtualization:** Oracle VirtualBox
- **Operating System:** Ubuntu Server 22.04.5 LTS
- **SIEM Tool:** Wazuh 4.12.0
- **SSH Client:** PuTTY
- **Browser:** Mozilla Firefox

---

## Lab Steps

### 1. Download Ubuntu Server ISO
![Ubuntu Download](screenshots/1.png)

### 2. Create Virtual Machine in VirtualBox
![VM Creation](screenshots/2.png)
![Hardware Settings](screenshots/3.png)
![Disk Settings](screenshots/4.png)

### 3. Ubuntu Server Installation
![GRUB Menu](screenshots/5.png)
![Language Selection](screenshots/6.png)
![Installation Type](screenshots/7.png)
![Network Configuration](screenshots/8.png)
![Proxy Configuration](screenshots/9.png)
![Mirror Configuration](screenshots/10.png)
![Storage Configuration](screenshots/11.png)
![Storage Summary](screenshots/12.png)

### 4. Post Installation & PuTTY Connection
![Ubuntu Login](screenshots/13.png)
![System Information](screenshots/23.png)
![PuTTY Configuration](screenshots/22.png)

### 5. Wazuh Installation
![Installing Java](screenshots/14.png)
![Downloading Wazuh Installer](screenshots/15.png)
![Wazuh Installation Process](screenshots/16.png)
![Installation Completed](screenshots/17.png)

### 6. Network & Access
![IP Address](screenshots/18.png)

### 7. Wazuh Dashboard Access
![Security Warning](screenshots/19.png)
![Wazuh Login Page](screenshots/20.png)
![Wazuh Dashboard Overview](screenshots/21.png)

### 8. Services Status
![Wazuh Manager Status](screenshots/24.png)
![Wazuh Indexer Status](screenshots/25.png)
![Wazuh Dashboard Status](screenshots/26.png)

---

## Key Achievements

- Successfully installed **Ubuntu Server 22.04.5 LTS** on VirtualBox
- Configured SSH access using **PuTTY**
- Installed **Wazuh 4.12.0** (All-in-One deployment)
- Wazuh Manager, Indexer, and Dashboard are **Active & Running**
- Successfully accessed Wazuh Web Dashboard

---

## Commands Used

- `ip addr show` / `hostname -I` → Check IP Address
- `sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh`
- `sudo bash wazuh-install.sh -a`
- `systemctl status wazuh-manager`
- `systemctl status wazuh-indexer`
- `systemctl status wazuh-dashboard`

---

## Conclusion

This lab successfully demonstrated the deployment of a full **Wazuh SIEM** stack on Ubuntu Server. The system is now ready for monitoring endpoints, detecting threats, and performing security analysis.

---

**Submitted by:**  
**Ragib Shahriar Abeg**  
**ID: 2304017**
