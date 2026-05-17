# 📋 Command Reference — Wazuh SIEM Lab
**Course:** SEC 204 — Threat Modelling and Security Monitoring Sessional  
**Student:** Ragib Shahriar Abeg · ID: 2304017

> A complete reference of all commands used in this lab, along with alternatives, variations, and advanced commands for future use.

---

## 📑 Table of Contents
1. [System & Network](#1--system--network)
2. [Package Management](#2--package-management)
3. [SSH & Remote Access](#3--ssh--remote-access)
4. [Wazuh Installation](#4--wazuh-installation)
5. [Wazuh Service Management](#5--wazuh-service-management)
6. [Wazuh Agent Management](#6--wazuh-agent-management)
7. [Wazuh Logs & Monitoring](#7--wazuh-logs--monitoring)
8. [Wazuh Configuration](#8--wazuh-configuration)
9. [OpenSearch / Indexer](#9--opensearch--indexer)
10. [Firewall & Ports](#10--firewall--ports)
11. [Disk & Storage](#11--disk--storage)
12. [User Management](#12--user-management)
13. [Future / Advanced Commands](#13--future--advanced-commands)

---

## 1 — System & Network

### Used in this lab
```bash
# Check all network interfaces and IP addresses
ip a

# Quick IP only
hostname -I

# Check system info on login (shown in MOTD)
# Displayed automatically — no command needed
```

### Alternatives & variations
```bash
# Older ifconfig (may need net-tools installed)
ifconfig

# Check specific interface
ip addr show enp0s3

# Check routing table
ip route

# Test connectivity
ping -c 4 8.8.8.8

# Check DNS resolution
nslookup google.com
dig google.com

# Check open ports on the system
ss -tuln
netstat -tuln          # older alternative (requires net-tools)

# Show hostname
hostname
hostnamectl

# Check system uptime
uptime

# Check OS version
lsb_release -a
cat /etc/os-release

# Check kernel version
uname -r
uname -a

# Check CPU info
lscpu
nproc

# Check RAM
free -h

# Check system load
top
htop                   # better UI (install: sudo apt install htop)
```

---

## 2 — Package Management

### Used in this lab
```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install Java JDK
sudo apt install default-jdk -y
```

### Alternatives & variations
```bash
# Full upgrade (handles dependency changes)
sudo apt full-upgrade -y

# Install specific package
sudo apt install <package-name> -y

# Remove a package
sudo apt remove <package-name>

# Remove package + config files
sudo apt purge <package-name>

# Remove unused dependencies
sudo apt autoremove -y

# Search for a package
apt search <keyword>

# Show package info
apt show <package-name>

# List installed packages
dpkg -l

# Check if a package is installed
dpkg -l | grep <package-name>

# Check Java version
java -version
javac -version

# List available Java versions
update-alternatives --list java
```

---

## 3 — SSH & Remote Access

### Used in this lab
```bash
# Connect via PuTTY (GUI — Windows)
# Host: 192.168.1.20  Port: 22  Type: SSH

# SSH from Linux/Mac terminal equivalent
ssh abeg@192.168.1.20
```

### Alternatives & variations
```bash
# SSH with specific port
ssh -p 22 abeg@192.168.1.20

# SSH with verbose output (useful for debugging)
ssh -v abeg@192.168.1.20

# SSH with key-based authentication (more secure)
ssh -i ~/.ssh/id_rsa abeg@192.168.1.20

# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id abeg@192.168.1.20

# Check SSH service status
sudo systemctl status ssh

# Restart SSH service
sudo systemctl restart ssh

# Check SSH config
sudo nano /etc/ssh/sshd_config

# Disable password auth (after setting up keys)
# In /etc/ssh/sshd_config:
# PasswordAuthentication no

# Check who is logged in via SSH
who
w
last

# Terminate an SSH session
exit
logout
```

---

## 4 — Wazuh Installation

### Used in this lab
```bash
# Download the Wazuh install script
sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh

# Run all-in-one installation
sudo bash wazuh-install.sh -a
```

### Alternatives & variations
```bash
# Download with progress visible
sudo curl -O https://packages.wazuh.com/4.12/wazuh-install.sh

# Verify script before running (good practice)
cat wazuh-install.sh | head -50

# All-in-one install with verbose output
sudo bash wazuh-install.sh -a -v

# Install with specific component only
sudo bash wazuh-install.sh --wazuh-server
sudo bash wazuh-install.sh --wazuh-indexer
sudo bash wazuh-install.sh --wazuh-dashboard

# Overwrite existing installation
sudo bash wazuh-install.sh -a --overwrite

# Check Wazuh version after install
/var/ossec/bin/wazuh-control info

# Get credentials after install (saved here)
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt

# Alternative: check install log
sudo cat /var/log/wazuh-install.log
```

---

## 5 — Wazuh Service Management

### Used in this lab
```bash
# Check Wazuh Manager status
systemctl status wazuh-manager --no-pager

# Check Wazuh Indexer status
systemctl status wazuh-indexer --no-pager

# Check Wazuh Dashboard status
systemctl status wazuh-dashboard --no-pager
```

### Alternatives & variations
```bash
# Start services
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard

# Stop services
sudo systemctl stop wazuh-manager
sudo systemctl stop wazuh-indexer
sudo systemctl stop wazuh-dashboard

# Restart services
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

# Enable on boot
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-indexer
sudo systemctl enable wazuh-dashboard

# Disable on boot
sudo systemctl disable wazuh-manager

# Check all Wazuh-related services at once
systemctl list-units | grep wazuh

# Using Wazuh's own control script (alternative to systemctl)
sudo /var/ossec/bin/wazuh-control start
sudo /var/ossec/bin/wazuh-control stop
sudo /var/ossec/bin/wazuh-control restart
sudo /var/ossec/bin/wazuh-control status
```

---

## 6 — Wazuh Agent Management

> These are **future commands** — for when you deploy agents on other machines to monitor them through this Wazuh server.

```bash
# On the AGENT machine — install Wazuh agent (Ubuntu/Debian)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent -y

# Configure agent to point to your Wazuh manager
sudo nano /var/ossec/etc/ossec.conf
# Set: <address>192.168.1.20</address>

# Register agent with manager (from manager side)
sudo /var/ossec/bin/manage_agents

# Or use agent-auth (from agent side)
sudo /var/ossec/bin/agent-auth -m 192.168.1.20

# Start agent
sudo systemctl start wazuh-agent
sudo systemctl enable wazuh-agent

# Check agent status on manager
sudo /var/ossec/bin/agent_control -l          # list all agents
sudo /var/ossec/bin/agent_control -s          # show agent summary
sudo /var/ossec/bin/agent_control -i 001      # info on agent 001

# Remove an agent
sudo /var/ossec/bin/manage_agents             # then choose Remove
```

---

## 7 — Wazuh Logs & Monitoring

### Used in this lab
```bash
# Verified via systemctl status output (shown in screenshots)
```

### Useful log commands
```bash
# Wazuh Manager main log
sudo tail -f /var/ossec/logs/ossec.log

# Wazuh alerts log (real-time alert stream)
sudo tail -f /var/ossec/logs/alerts/alerts.log

# Alerts in JSON format (better for parsing)
sudo tail -f /var/ossec/logs/alerts/alerts.json

# Wazuh Manager error log
sudo tail -f /var/ossec/logs/ossec.json

# Wazuh Indexer log
sudo tail -f /var/log/wazuh-indexer/wazuh-cluster.log

# Dashboard log
sudo journalctl -u wazuh-dashboard -f

# Manager log via journalctl
sudo journalctl -u wazuh-manager -f
sudo journalctl -u wazuh-manager --since "1 hour ago"
sudo journalctl -u wazuh-manager --no-pager -n 50

# Search alerts for specific rule
sudo grep "Rule ID: 5710" /var/ossec/logs/alerts/alerts.log

# Count alerts today
sudo grep "$(date +%Y %b %e)" /var/ossec/logs/ossec.log | wc -l
```

---

## 8 — Wazuh Configuration

> Key config files for future customization.

```bash
# Main Wazuh Manager config
sudo nano /var/ossec/etc/ossec.conf

# Wazuh rules directory
ls /var/ossec/ruleset/rules/

# Custom rules file (add your own rules here)
sudo nano /var/ossec/etc/rules/local_rules.xml

# Custom decoders
sudo nano /var/ossec/etc/decoders/local_decoder.xml

# Wazuh API config
sudo nano /var/ossec/api/configuration/api.yaml

# Dashboard config
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml

# Indexer config
sudo nano /etc/wazuh-indexer/opensearch.yml

# After editing config — validate and restart
sudo /var/ossec/bin/wazuh-control restart

# Check config for syntax errors
sudo /var/ossec/bin/ossec-logtest
```

---

## 9 — OpenSearch / Indexer

```bash
# Check indexer/OpenSearch health
curl -k -u admin:<password> https://192.168.1.20:9200/_cluster/health?pretty

# List all indices
curl -k -u admin:<password> https://192.168.1.20:9200/_cat/indices?v

# Check node info
curl -k -u admin:<password> https://192.168.1.20:9200/_nodes?pretty

# Check disk usage by index
curl -k -u admin:<password> https://192.168.1.20:9200/_cat/indices?v&s=store.size:desc

# Delete old indices (free up space — important, indexer fills disk fast)
curl -k -u admin:<password> -X DELETE https://192.168.1.20:9200/wazuh-alerts-4.x-2026.05.01

# Check OpenSearch version
curl -k -u admin:<password> https://192.168.1.20:9200
```

---

## 10 — Firewall & Ports

```bash
# Check UFW status
sudo ufw status verbose

# Allow SSH
sudo ufw allow 22/tcp

# Allow Wazuh dashboard (HTTPS)
sudo ufw allow 443/tcp

# Allow Wazuh agent communication
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp

# Allow Wazuh agent enrollment
sudo ufw allow 1515/tcp

# Allow Wazuh API
sudo ufw allow 55000/tcp

# Allow OpenSearch
sudo ufw allow 9200/tcp

# Enable UFW
sudo ufw enable

# Check which process is using a port
sudo ss -tulnp | grep 443
sudo lsof -i :443

# Wazuh default ports reference:
# 1514  — Agent communication (UDP/TCP)
# 1515  — Agent enrollment
# 1516  — Wazuh cluster
# 55000 — Wazuh REST API
# 9200  — OpenSearch REST API
# 9300  — OpenSearch cluster
# 443   — Wazuh Dashboard (HTTPS)
```

---

## 11 — Disk & Storage

> Critical for Wazuh — the indexer fills disk quickly with alert data.

```bash
# Check disk usage
df -h

# Check disk usage of specific directory
du -sh /var/ossec/logs/
du -sh /var/lib/wazuh-indexer/

# Find largest files
sudo find /var/ossec -type f -size +100M

# Check inode usage (can fill up separately from disk)
df -i

# Clear old Wazuh logs manually
sudo find /var/ossec/logs/alerts/ -name "*.log" -mtime +30 -delete

# Check wazuh-indexer data directory size
du -sh /var/lib/wazuh-indexer/nodes/
```

---

## 12 — User Management

```bash
# Add a new user
sudo adduser <username>

# Add user to sudo group
sudo usermod -aG sudo <username>

# Switch to another user
su - <username>

# Check current user
whoami

# Change password
passwd
sudo passwd <username>

# Lock a user account
sudo usermod -L <username>

# Unlock a user account
sudo usermod -U <username>

# Delete a user
sudo deluser <username>

# Change Wazuh API admin password
sudo /var/ossec/bin/wazuh-passwords-tool.sh -u wazuh -p <newpassword>

# Change Wazuh dashboard admin password
sudo /var/ossec/bin/wazuh-passwords-tool.sh -u admin -p <newpassword>
```

---

## 13 — Future / Advanced Commands

> Commands you will likely use as you extend this lab — agent deployment, threat hunting, API usage, and NetGuard integration.

```bash
# ── Wazuh REST API (useful for NetGuard integration) ──────────────────

# Get auth token
TOKEN=$(curl -s -u wazuh:wazuh -k -X GET "https://192.168.1.20:55000/security/user/authenticate?raw=true")

# List all agents via API
curl -k -X GET "https://192.168.1.20:55000/agents" -H "Authorization: Bearer $TOKEN"

# Get recent alerts via API
curl -k -X GET "https://192.168.1.20:55000/overview/agents" -H "Authorization: Bearer $TOKEN"

# Get specific agent info
curl -k -X GET "https://192.168.1.20:55000/agents/001" -H "Authorization: Bearer $TOKEN"


# ── File Integrity Monitoring (FIM) ───────────────────────────────────

# Trigger FIM scan manually
sudo /var/ossec/bin/agent_control -r -u 001

# View FIM database
sudo sqlite3 /var/ossec/queue/fim/db/fim.db "SELECT * FROM fim_entry LIMIT 20;"


# ── Active Response ───────────────────────────────────────────────────

# Manually trigger active response (e.g. block an IP)
sudo /var/ossec/bin/agent_control -b 192.168.1.100 -f firewall-drop0 -u 001


# ── Vulnerability Detection ───────────────────────────────────────────

# Trigger vulnerability scan
sudo /var/ossec/bin/wazuh-modulesd --scan-on-start


# ── Backup & Restore ─────────────────────────────────────────────────

# Backup Wazuh config
sudo tar -cvf wazuh-backup.tar /var/ossec/etc/

# Backup agent keys
sudo cp /var/ossec/etc/client.keys ~/client.keys.bak


# ── Useful one-liners ─────────────────────────────────────────────────

# Count total alerts in current log
sudo wc -l /var/ossec/logs/alerts/alerts.log

# Find failed SSH login alerts
sudo grep "authentication failure" /var/ossec/logs/alerts/alerts.log | tail -20

# Monitor Wazuh manager in real time
sudo journalctl -u wazuh-manager -f --output=short-precise

# Check Wazuh indexer cluster health quickly
curl -sk -u admin:<password> https://localhost:9200/_cluster/health | python3 -m json.tool

# Restart entire Wazuh stack in order
sudo systemctl restart wazuh-indexer && sleep 10 && sudo systemctl restart wazuh-manager && sleep 5 && sudo systemctl restart wazuh-dashboard
```

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 204  
[![GitHub](https://img.shields.io/badge/GitHub-rsabeg-181717?style=flat-square&logo=github)](https://github.com/rsabeg)

</div>

