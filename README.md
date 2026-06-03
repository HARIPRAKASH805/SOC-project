## 👤 Author

**HARIPRAKASH**
- GitHub: [HARIPRAKASH](https://github.com/HARIPRAKASH805)
- Project Link: [SOC Detection Lab](https://github.com/HARIPRAKASH805/SOC-project)
# 🔐 SOC Detection Lab – SSH Brute Force Attack Detection

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Splunk](https://img.shields.io/badge/Splunk-9.2.0-green)](https://www.splunk.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT&CK-T1110.001-red)](https://attack.mitre.org/techniques/T1110/001/)

## 🎯 Project Overview

A hands-on Security Operations Center (SOC) lab demonstrating SSH brute-force attack detection using Splunk SIEM.

### What This Lab Does
- ✅ Simulates SSH brute-force attacks using Hydra
- ✅ Collects and analyzes authentication logs in Splunk
- ✅ Detects attack patterns using SPL queries
- ✅ Maps findings to MITRE ATT&CK framework

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────┐
│         KALI LINUX (Attacker)           │
│  ┌────────────────────────────────┐     │
│  │    Splunk Enterprise + Hydra    │     │
│  └────────────────────────────────┘     │
│              │                           │
│              │ SSH Brute Force           │
│              ▼                           │
├─────────────────────────────────────────┤
│         UBUNTU 22.04 (Target)           │
│  ┌────────────────────────────────┐     │
│  │    SSH Server (port 22)         │     │
│  │    /var/log/auth.log            │     │
│  └────────────────────────────────┘     │
└─────────────────────────────────────────┘
```

## 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Splunk Enterprise | 9.2.0 | SIEM platform |
| Kali Linux | 2024.1 | Attack machine |
| Hydra | 9.4 | Brute force tool |
| Ubuntu | 22.04 LTS | Target victim |
| VirtualBox | 7.0 | Virtualization |

## 🔍 Detection Queries

### Basic Detection (10+ failures in 5 minutes)
```spl
index=main "Failed password" 
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| stats count by src_ip 
| where count > 10
```

### Advanced Pattern Detection
```spl
index=main "Failed password" 
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10
| sort - count
```

### Multiple Username Attempts
```spl
index=main "Failed password for invalid user"
| rex "invalid user (?<username>\w+)"
| stats dc(username) as unique_users by src_ip
| where unique_users > 5
```

## 🎯 MITRE ATT&CK Mapping

| Attribute | Value |
|-----------|-------|
| **Tactic** | TA0006 - Credential Access |
| **Technique ID** | T1110 - Brute Force |
| **Sub-technique** | T1110.001 - Password Guessing |
| **Data Source** | Authentication logs |

## 📊 Lab Results

| Metric | Result |
|--------|--------|
| Attack speed | 500+ attempts/minute |
| Detection threshold | 10 failures/5 minutes |
| Time to alert | < 5 minutes |
| False positives | Minimal with proper tuning |

## 📸 Screenshots

*[You'll add your screenshots here after uploading]*

## 🚀 Quick Setup Steps

1. Install VirtualBox on your computer
2. Create Kali Linux and Ubuntu VMs
3. Install Splunk on Kali
4. Configure Ubuntu to forward auth.log to Splunk
5. Run Hydra attack from Kali
6. Search logs in Splunk using queries above

## ⚠️ Disclaimer

> **Educational purpose only** - This lab is designed for isolated virtual environments. Never attack systems without permission.

## 👤 Author

**Your Name**
- GitHub: [your-username](https://github.com/your-username)
- Project Link: [SOC Detection Lab](https://github.com/your-username/SOC-Detection-Lab)

---
⭐ Star this repository if you found it helpful!
