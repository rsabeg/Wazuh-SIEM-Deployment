<div align="center">

# 🔧 Troubleshooting Guide — Wazuh SIEM Deployment

</div>

> Real issues encountered during this lab and their solutions, plus common problems you are likely to hit when working with Wazuh on Ubuntu Server.

---

## Table of Contents
1. [Disk Space Exhaustion](#1--disk-space-exhaustion)
2. [Browser TLS Certificate Warning](#2--browser-tls-certificate-warning)
3. [Wazuh Service Won't Start](#3--wazuh-service-wont-start)
4. [Dashboard Not Loading](#4--dashboard-not-loading)
5. [SSH Connection Refused](#5--ssh-connection-refused)
6. [Agent Not Connecting to Manager](#6--agent-not-connecting-to-manager)
7. [Wazuh Indexer Java Out of Memory](#7--wazuh-indexer-java-out-of-memory)
8. [Installation Script Fails](#8--installation-script-fails)
9. [Forgot Admin Password](#9--forgot-admin-password)
10. [High CPU / Memory Usage](#10--high-cpu--memory-usage)

---

## 1 — Disk Space Exhaustion

**Observed in this lab.** After installing Wazuh, disk usage jumped to **85.8% of 22.47 GB**. The Wazuh Indexer log showed:

```
Caused by: java.io.IOException: No space left on device
```

### Diagnosis
```bash
# Check overall disk usage
df -h

# Find what's consuming space
du -sh /var/lib/wazuh-indexer/
du -sh /var/ossec/logs/
du -sh /var/log/
```

### Solution — Short Term (Free up space now)
```bash
# Delete old alert logs (keep last 7 days)
sudo find /var/ossec/logs/alerts/ -name "*.log" -mtime +7 -delete
sudo find /var/ossec/logs/alerts/ -name "*.json" -mtime +7 -delete

# Clean apt cache
sudo apt clean
sudo apt autoremove -y

# Remove old journal logs
sudo journalctl --vacuum-time=3d
```

### Solution — Long Term (Prevent recurrence)
Configure Index Lifecycle Management in OpenSearch to auto-delete old indices:

```bash
# Get current indices
curl -sk -u admin:<password> https://localhost:9200/_cat/indices?v

# Delete indices older than specific date (replace date)
curl -sk -u admin:<password> -X DELETE \
  "https://localhost:9200/wazuh-alerts-4.x-2026.05.*"
```

Or increase VM disk size before installation — **50 GB minimum** is recommended for any non-trivial Wazuh deployment.

---

## 2 — Browser TLS Certificate Warning

**Observed in this lab.** Firefox showed:

```
Warning: Potential Security Risk Ahead
SEC_ERROR_UNKNOWN_ISSUER
```

### Why This Happens
Wazuh generates a **self-signed TLS certificate** during installation. Browsers don't trust self-signed certs by default because there's no Certificate Authority (CA) backing them.

### Solution — For Lab Use (Bypass the warning)
1. On the Firefox warning page, click **Advanced**
2. Click **Accept the Risk and Continue**
3. You will be redirected to the Wazuh login page

This is safe in a controlled lab environment since you know exactly what server you're connecting to.

### Solution — For Production Use (Proper fix)
Replace the self-signed cert with a CA-signed certificate:

```bash
# Using Let's Encrypt (requires a public domain)
sudo apt install certbot -y
sudo certbot certonly --standalone -d yourdomain.com

# Copy certs to Wazuh Dashboard
sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem \
  /etc/wazuh-dashboard/certs/dashboard.pem
sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem \
  /etc/wazuh-dashboard/certs/dashboard-key.pem

sudo systemctl restart wazuh-dashboard
```

---

## 3 — Wazuh Service Won't Start

### Diagnosis
```bash
# Check which service is failing
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager

# Check detailed logs
sudo journalctl -u wazuh-manager -n 50 --no-pager
sudo journalctl -u wazuh-indexer -n 50 --no-pager
```

### Common Causes & Fixes

**Cause: Config file syntax error**
```bash
# Test config before restarting
sudo /var/ossec/bin/ossec-logtest

# Check XML syntax of ossec.conf
xmllint --noout /var/ossec/etc/ossec.conf
```

**Cause: Port already in use**
```bash
# Find what's using the port
sudo ss -tulnp | grep 1514
sudo lsof -i :9200

# Kill conflicting process if safe to do so
sudo kill -9 <PID>
```

**Cause: Insufficient permissions**
```bash
# Fix Wazuh directory permissions
sudo chown -R ossec:ossec /var/ossec/
sudo systemctl restart wazuh-manager
```

**Cause: Restart order matters**
Always restart in this order — Indexer must be up before Manager and Dashboard:
```bash
sudo systemctl restart wazuh-indexer
sleep 15
sudo systemctl restart wazuh-manager
sleep 5
sudo systemctl restart wazuh-dashboard
```

---

## 4 — Dashboard Not Loading

### Symptoms
- Browser times out on `https://192.168.1.20`
- "Connection refused" error
- Page loads but shows blank screen

### Diagnosis
```bash
# Check dashboard service
sudo systemctl status wazuh-dashboard --no-pager

# Check if port 443 is listening
sudo ss -tulnp | grep 443

# Check dashboard logs
sudo journalctl -u wazuh-dashboard -n 100 --no-pager
```

### Common Fixes

**Fix 1: Dashboard service not running**
```bash
sudo systemctl start wazuh-dashboard
sudo systemctl enable wazuh-dashboard
```

**Fix 2: Indexer not ready yet (Dashboard depends on it)**
```bash
# Check indexer health first
curl -sk -u admin:<password> https://localhost:9200/_cluster/health?pretty

# If indexer is red/unhealthy, restart it first
sudo systemctl restart wazuh-indexer
sleep 20
sudo systemctl restart wazuh-dashboard
```

**Fix 3: Firewall blocking port 443**
```bash
sudo ufw allow 443/tcp
sudo ufw reload
```

**Fix 4: Wrong URL — must use HTTPS not HTTP**
```
❌ http://192.168.1.20
✅ https://192.168.1.20
```

---

## 5 — SSH Connection Refused

### Symptoms
PuTTY shows: `Network error: Connection refused`

### Diagnosis
```bash
# On the VM console — check if SSH is running
sudo systemctl status ssh

# Check if port 22 is listening
sudo ss -tulnp | grep 22

# Check firewall
sudo ufw status
```

### Fixes

**Fix 1: SSH service not running**
```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

**Fix 2: SSH not installed**
```bash
sudo apt install openssh-server -y
sudo systemctl start ssh
```

**Fix 3: Firewall blocking SSH**
```bash
sudo ufw allow 22/tcp
sudo ufw reload
```

**Fix 4: Wrong IP address**
```bash
# Confirm VM IP from the console
ip a
# Look for inet under enp0s3
```

**Fix 5: VirtualBox network mode issue**
If using NAT mode, you need port forwarding. Switch to **Host-Only Adapter** or **Bridged Adapter** in VirtualBox VM settings for direct IP access.

---

## 6 — Agent Not Connecting to Manager

### Symptoms
Agent shows as `Never connected` or `Disconnected` in the Dashboard.

### Diagnosis
```bash
# On Manager — check agent list
sudo /var/ossec/bin/agent_control -l

# On Agent — check agent service
sudo systemctl status wazuh-agent

# On Agent — check agent log
sudo tail -f /var/ossec/logs/ossec.log
```

### Common Fixes

**Fix 1: Wrong manager IP in agent config**
```bash
# On the agent machine
sudo nano /var/ossec/etc/ossec.conf
# Verify: <address>192.168.1.20</address>
sudo systemctl restart wazuh-agent
```

**Fix 2: Firewall blocking agent port on manager**
```bash
# On the manager
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp
sudo ufw allow 1515/tcp
sudo ufw reload
```

**Fix 3: Agent not registered**
```bash
# Re-register agent on manager
sudo /var/ossec/bin/manage_agents
# Choose A to add, then enter agent details
```

---

## 7 — Wazuh Indexer Java Out of Memory

### Symptoms
Indexer log shows:
```
java.lang.OutOfMemoryError: Java heap space
```

### Diagnosis
```bash
sudo journalctl -u wazuh-indexer -n 100 | grep -i "memory\|heap\|OOM"
free -h
```

### Fix — Increase JVM Heap Size
```bash
sudo nano /etc/wazuh-indexer/jvm.options

# Find and modify these lines:
-Xms1g   # change to -Xms2g  (minimum heap)
-Xmx1g   # change to -Xmx2g  (maximum heap)

# Rule: never exceed 50% of total RAM for heap
# With 5.7GB RAM: max heap = ~2.5GB

sudo systemctl restart wazuh-indexer
```

---

## 8 — Installation Script Fails

### Common Failure Points

**Failure: Hardware requirements not met**
```
ERROR: Your system does not meet the minimum hardware requirements
```
Wazuh requires at minimum 4 GB RAM and 2 CPUs. Verify:
```bash
free -h       # check RAM
nproc         # check CPUs
```
Increase VM RAM to at least 4096 MB in VirtualBox settings.

**Failure: Network unreachable during install**
```bash
# Test connectivity
ping -c 3 packages.wazuh.com

# Check DNS
nslookup packages.wazuh.com

# If using proxy — set it
export http_proxy=http://proxy:port
export https_proxy=http://proxy:port
```

**Failure: Previous failed install blocking reinstall**
```bash
# Force overwrite existing installation
sudo bash wazuh-install.sh -a --overwrite
```

**Failure: Partially downloaded installer**
```bash
# Re-download fresh copy
rm wazuh-install.sh
sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

---

## 9 — Forgot Admin Password

### Retrieve from Installation Files
```bash
# Credentials are saved here immediately after install
sudo tar -O -xvf wazuh-install-files.tar \
  wazuh-install-files/wazuh-passwords.txt
```

### Reset Password
```bash
# Use Wazuh password tool
sudo /var/ossec/bin/wazuh-passwords-tool.sh -u admin -p NewPassword123!

# Restart services after password change
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-dashboard
```

---

## 10 — High CPU / Memory Usage

### Diagnosis
```bash
# See which Wazuh process is consuming most
top -p $(pgrep -d',' -f wazuh)
htop

# Check per-service memory
systemctl status wazuh-indexer --no-pager | grep Memory
systemctl status wazuh-manager --no-pager | grep Memory
systemctl status wazuh-dashboard --no-pager | grep Memory
```

### Common Causes & Fixes

**Cause: Indexer initial startup (normal)**
OpenSearch takes 5–10 minutes to fully initialize and will spike CPU during this time. Wait and recheck.

**Cause: Too many agents sending high event volumes**
```bash
# Check event rate per agent
sudo /var/ossec/bin/agent_control -s

# Set rate limits in ossec.conf
# <global>
#   <max_output_size>25165824</max_output_size>
# </global>
```

**Cause: Log file too large causing collector bottleneck**
```bash
sudo du -sh /var/ossec/logs/alerts/alerts.log
# If >1GB, rotate it
sudo logrotate -f /etc/logrotate.d/wazuh
```

---

<div align="center">

**Ragib Shahriar Abeg** · ID: 2304017 · SEC 204  
[![GitHub](https://img.shields.io/badge/GitHub-ICE1945-181717?style=flat-square&logo=github)](https://github.com/ICE1945)

</div>

