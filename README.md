# Wazuh SIEM Installation on Ubuntu Server 22.04

**Lab Report**  
**Course:** Threat Modelling and Security Monitoring Sessional (SEC 204)  
**Student Name:** Ragib Shahriar Abeg  
**Student ID:** 2304017  

---

## 📋 Objective
To install and configure **Wazuh** (Open Source Security Monitoring Platform) on **Ubuntu Server 22.04 LTS** using VirtualBox and access it via PuTTY and Web Dashboard.

---

## 🛠️ Tools & Technologies Used

- **Virtualization:** Oracle VirtualBox
- **Operating System:** Ubuntu Server 22.04.5 LTS
- **SIEM Tool:** Wazuh 4.12.0
- **SSH Client:** PuTTY
- **Browser:** Mozilla Firefox

---

## 📸 Lab Steps

### 1. Download Ubuntu Server ISO
![Ubuntu Download](screenshots/1.png)

### 2. Create Virtual Machine in VirtualBox
![VM Creation](Screenshots/2.png)
![Hardware Settings](Screenshots/3.png)
![Disk Settings](Screenshots/4.png)

### 3. Ubuntu Server Installation
![GRUB Menu](Screenshots/5.png)
![Language Selection](Screenshots/6.png)
![Installation Type](Screenshots/7.png)
![Network Configuration](Screenshots/8.png)
![Proxy Configuration](Screenshots/9.png)
![Mirror Configuration](Screenshots/10.png)
![Storage Configuration](Screenshots/11.png)
![Storage Summary](Screenshots/12.png)

### 4. Post Installation & PuTTY Connection
![Ubuntu Login](Screenshots/13.png)
![System Information](Screenshots/23.png)
![PuTTY Configuration](Screenshots/22.png)

### 5. Wazuh Installation
![Installing Java](Screenshots/14.png)
![Downloading Wazuh Installer](Screenshots/15.png)
![Wazuh Installation Process](Screenshots/16.png)
![Installation Completed](Screenshots/17.png)

### 6. Network & Access
![IP Address](Screenshots/18.png)

### 7. Wazuh Dashboard Access
![Security Warning](Screenshots/19.png)
![Wazuh Login Page](Screenshots/20.png)
![Wazuh Dashboard Overview](Screenshots/21.png)

### 8. Services Status
![Wazuh Manager Status](Screenshots/24.png)
![Wazuh Indexer Status](Screenshots/25.png)
![Wazuh Dashboard Status](Screenshots/26.png)

---

## ✅ Key Achievements

- Successfully installed **Ubuntu Server 22.04.5 LTS** on VirtualBox
- Configured SSH access using **PuTTY**
- Installed **Wazuh 4.12.0** (All-in-One deployment)
- Wazuh Manager, Indexer, and Dashboard are **Active & Running**
- Successfully accessed Wazuh Web Dashboard

---

## 🔧 Commands Used

- `ip addr show` / `hostname -I` → Check IP Address
- `sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh`
- `sudo bash wazuh-install.sh -a`
- `systemctl status wazuh-manager`
- `systemctl status wazuh-indexer`
- `systemctl status wazuh-dashboard`

---

## 📝 Conclusion

This lab successfully demonstrated the deployment of a full **Wazuh SIEM** stack on Ubuntu Server. The system is now ready for monitoring endpoints, detecting threats, and performing security analysis.

---

**Submitted by:**  
**Ragib Shahriar Abeg**  
**ID: 2304017**  
**SEC 204 - Threat Modelling and Security Monitoring Sessional**
