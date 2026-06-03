# 🚀 Complete Setup Guide

> Follow this guide step-by-step to replicate the SOC Detection Lab from scratch. Estimated time: **2–3 hours**.

---

## 📋 Prerequisites

- A computer with at least **8GB RAM** and **50GB free disk space**
- [VirtualBox 7.0](https://www.virtualbox.org/wiki/Downloads) installed
- Downloaded ISO files:
  - [Kali Linux 2024.1](https://www.kali.org/get-kali/)
  - [Ubuntu 22.04 LTS Server](https://ubuntu.com/download/server)
- [Splunk Enterprise 9.2.0](https://www.splunk.com/en_us/download/splunk-enterprise.html) (free trial — requires account)
- [Splunk Universal Forwarder 9.2.0](https://www.splunk.com/en_us/download/universal-forwarder.html)

---

## 🖥️ Step 1 — Create Virtual Machines in VirtualBox

### VM 1: Kali Linux (Attacker)

| Setting | Value |
|---------|-------|
| Name | `Kali-Attacker` |
| RAM | 2048 MB |
| Disk | 20 GB |
| Network | Host-Only Adapter |

### VM 2: Ubuntu 22.04 (Target)

| Setting | Value |
|---------|-------|
| Name | `Ubuntu-Target` |
| RAM | 2048 MB |
| Disk | 20 GB |
| Network | Host-Only Adapter |

### Network Configuration

Both VMs must be on the same **Host-Only Network** so they can communicate but stay isolated from the internet.

In VirtualBox:
1. Go to **File → Host Network Manager**
2. Create a new network: `192.168.56.0/24`
3. Set each VM's **Network Adapter → Host-Only Adapter → vboxnet0**

After booting, verify IPs:
```bash
# On each VM:
ip addr show
# You should see an IP like 192.168.56.101 or 192.168.56.102
```

---

## 🎯 Step 2 — Configure Ubuntu (Target Machine)

### 2a. Install SSH Server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh   # Should show: active (running)
```

### 2b. Verify SSH is Listening

```bash
ss -tlnp | grep 22
# Output should show: 0.0.0.0:22
```

### 2c. Create Test User Accounts (for the attack to target)

```bash
sudo useradd -m testuser
sudo passwd testuser
# Set a simple password like: password123
```

### 2d. Note the Ubuntu IP Address

```bash
hostname -I
# Example output: 192.168.56.102
# This is your TARGET_IP — remember it
```

---

## 📦 Step 3 — Install Splunk Enterprise (on your Host or a 3rd VM)

> **Important:** Splunk runs on the **monitoring machine** — NOT on Kali. In this lab, you can run it directly on your host computer.

### 3a. Install Splunk (Linux/Mac)

```bash
# Download the .deb or .rpm from splunk.com, then:
sudo dpkg -i splunk-9.2.0-*.deb

# Start Splunk
sudo /opt/splunk/bin/splunk start --accept-license

# Enable auto-start on boot
sudo /opt/splunk/bin/splunk enable boot-start
```

### 3b. Access Splunk Web UI

Open your browser and go to:
```
http://localhost:8000
```
- Default username: `admin`
- Password: whatever you set during first start

### 3c. Create an Index for SSH Logs

In Splunk Web:
1. Go to **Settings → Indexes → New Index**
2. Name: `main` (or use the existing default `main` index)
3. Click Save

### 3d. Enable Splunk to Receive Forwarded Logs

```
Settings → Forwarding and Receiving → Configure Receiving → New Receiving Port
Port: 9997
```

---

## 📡 Step 4 — Install Splunk Universal Forwarder on Ubuntu

> The Universal Forwarder is a lightweight agent that ships logs from Ubuntu to your Splunk SIEM.

### 4a. Install the Forwarder

```bash
# On Ubuntu VM — download and install
wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/releases/9.2.0/linux/splunkforwarder-9.2.0-<build>-linux-2.6-amd64.deb"
sudo dpkg -i splunkforwarder.deb
```

### 4b. Start and Configure the Forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes \
  --no-prompt --seed-passwd YourPassword123

# Point the forwarder to your Splunk server
# Replace SPLUNK_SERVER_IP with your host's IP (e.g., 192.168.56.1)
sudo /opt/splunkforwarder/bin/splunk add forward-server SPLUNK_SERVER_IP:9997 \
  -auth admin:YourPassword123
```

### 4c. Configure the Forwarder to Monitor auth.log

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log \
  -index main \
  -sourcetype linux_secure \
  -auth admin:YourPassword123
```

### 4d. Restart the Forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

### 4e. Verify Logs are Flowing

In Splunk Web, run this search:
```spl
index=main sourcetype=linux_secure
```
You should see Ubuntu auth logs appearing. If not, check that port 9997 is open on your Splunk server.

---

## ⚔️ Step 5 — Run the Brute Force Attack from Kali

### 5a. Install Hydra (if not already installed)

```bash
# On Kali Linux:
sudo apt update
sudo apt install hydra -y
```

### 5b. Prepare a Password List

```bash
# Kali comes with rockyou.txt — extract it first:
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Or create a small custom list for testing:
echo -e "password\n123456\nadmin\nletmein\ntestpass\npassword123" > test-passwords.txt
```

### 5c. Run the Hydra Attack

```bash
# Replace 192.168.56.102 with your Ubuntu VM's IP
# -l  = single username to try
# -P  = password list file
# -t  = number of parallel threads (4 is safe; 16 is fast)
# -V  = verbose (show each attempt)

hydra -l testuser -P test-passwords.txt ssh://192.168.56.102 -t 4 -V

# To try multiple usernames:
hydra -L /usr/share/wordlists/metasploit/unix_users.txt \
      -P test-passwords.txt \
      ssh://192.168.56.102 -t 4
```

### 5d. What to Expect

- Hydra will show each attempt: `[22][ssh] host: 192.168.56.102 login: testuser password: password`
- When it finds the correct password, it shows: `[22][ssh] host: 192.168.56.102 login: testuser password: password123`
- Meanwhile, Ubuntu's `/var/log/auth.log` is filling up with failed login entries

---

## 🔎 Step 6 — Detect the Attack in Splunk

### 6a. Open Splunk and Run Detection Queries

Go to `http://localhost:8000` → **Search & Reporting**

Run the basic detection query:
```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10
| sort - count
```

You should see Kali's IP address with a high count.

### 6b. Set Up Alerts

See [`alerts.md`](./alerts.md) for full alert configuration steps in Splunk.

---

## ✅ Verification Checklist

- [ ] Ubuntu SSH server is running on port 22
- [ ] Splunk Universal Forwarder is installed on Ubuntu
- [ ] auth.log events appear in Splunk (`index=main sourcetype=linux_secure`)
- [ ] Hydra successfully runs from Kali and generates failed login events
- [ ] SPL detection query returns Kali's IP with a high failure count
- [ ] Alert fires when threshold (10 failures in 5 min) is crossed

---

## 🛠️ Troubleshooting

| Problem | Solution |
|---------|---------|
| No logs in Splunk | Check forwarder is running: `splunk status`; check port 9997 is open |
| SSH connection refused from Kali | Verify Ubuntu SSH is running: `systemctl status ssh` |
| Hydra gets 0 results | Check network connectivity: `ping TARGET_IP` from Kali |
| auth.log not updating | Try `sudo tail -f /var/log/auth.log` and SSH manually to generate events |
| Splunk shows wrong sourcetype | Re-add monitor with `-sourcetype linux_secure` explicitly |
