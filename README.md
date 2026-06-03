# 🔐 SOC Detection Lab – SSH Brute Force Attack Detection

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Splunk](https://img.shields.io/badge/Splunk-9.2.0-green)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110.001-red)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## 🎯 Project Overview

A hands-on Security Operations Center (SOC) lab demonstrating real-world SSH brute-force attack **simulation**, **log collection**, and **detection** using Splunk SIEM. This project maps directly to the MITRE ATT&CK framework and follows real SOC analyst workflows.

### What This Lab Does

- ✅ Simulates SSH brute-force attacks using Hydra (from Kali Linux)
- ✅ Collects and forwards authentication logs to Splunk via Universal Forwarder
- ✅ Detects attack patterns using SPL (Splunk Processing Language) queries
- ✅ Fires real-time alerts when thresholds are crossed
- ✅ Maps all findings to MITRE ATT&CK framework (T1110.001)

---

## 🏗️ Lab Architecture

```
┌──────────────────────────────────────────────┐
│           KALI LINUX (Attacker VM)            │
│   ┌────────────────────────────────────┐      │
│   │         Hydra (Brute Force)         │      │
│   └────────────────────────────────────┘      │
│                     │                          │
│           SSH Brute Force (port 22)            │
│                     ▼                          │
├──────────────────────────────────────────────┤
│          UBUNTU 22.04 (Target VM)             │
│   ┌────────────────────────────────────┐      │
│   │   SSH Server (port 22)             │      │
│   │   /var/log/auth.log                │      │
│   │   Splunk Universal Forwarder       │      │
│   └──────────────┬─────────────────────┘      │
│                  │ Forwards logs               │
│                  ▼                             │
├──────────────────────────────────────────────┤
│        SPLUNK SIEM (Monitoring VM or Host)    │
│   ┌────────────────────────────────────┐      │
│   │   Splunk Enterprise 9.2.0          │      │
│   │   Dashboards + Alerts + SPL        │      │
│   └────────────────────────────────────┘      │
└──────────────────────────────────────────────┘
```

> **Note:** Splunk Enterprise runs on a dedicated monitoring machine (or your host OS), NOT on the Kali attacker machine. Kali is used only for attack simulation.

---

## 🛠️ Tools Used

| Tool                      | Version   | Purpose                        |
| ------------------------- | --------- | ------------------------------ |
| Splunk Enterprise         | 9.2.0     | SIEM platform (log analysis)   |
| Splunk Universal Forwarder| 9.2.0     | Ships logs from Ubuntu → Splunk|
| Kali Linux                | 2024.1    | Attack machine                 |
| Hydra                     | 9.4       | SSH brute-force tool           |
| Ubuntu                    | 22.04 LTS | Target/victim machine          |
| VirtualBox                | 7.0       | Virtualization platform        |

---

## 🔍 Detection Queries

See full query library → [`detection-queries.md`](./detection-queries.md)

### Quick Reference — Basic Detection (10+ failures in 5 min)

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10
| sort - count
```

---

## 🎯 MITRE ATT&CK Mapping

See full mapping → [`MITRE-MAPPING.md`](./MITRE-MAPPING.md)

| Attribute         | Value                              |
| ----------------- | ---------------------------------- |
| **Tactic**        | TA0006 - Credential Access         |
| **Technique**     | T1110 - Brute Force                |
| **Sub-technique** | T1110.001 - Password Guessing      |
| **Also covers**   | T1021.004 (SSH), T1078 (Valid Accts)|
| **Data Source**   | Authentication logs (auth.log)     |

---

## 📊 Lab Results

| Metric               | Result                      |
| -------------------- | --------------------------- |
| Attack speed         | 500+ attempts/minute        |
| Detection threshold  | 10 failures / 5 minutes     |
| Time to alert        | < 5 minutes                 |
| False positive rate  | Minimal with sourcetype filter |

---

## 📸 Screenshots

> Add your screenshots to a `/screenshots` folder and link them below.

| What | Screenshot |
|------|-----------|
| Splunk Dashboard – Attack Detected | `screenshots/splunk-dashboard.png` |
| SPL Query Results | `screenshots/spl-query-results.png` |
| Hydra Running (Kali Terminal) | `screenshots/hydra-attack.png` |
| Alert Firing in Splunk | `screenshots/alert-triggered.png` |

---

## 🚀 Quick Start

See the full guide → [`SETUP-GUIDE.md`](./SETUP-GUIDE.md)

1. Install VirtualBox and create **3 VMs**: Kali, Ubuntu, and Splunk (or use host for Splunk)
2. Set all VMs to **Host-Only Network** adapter so they can talk to each other
3. Install **Splunk Enterprise** on your monitoring machine
4. Install **Splunk Universal Forwarder** on Ubuntu and point it at `/var/log/auth.log`
5. Run **Hydra** brute-force from Kali against Ubuntu's SSH port
6. Open **Splunk** and run the SPL detection queries

---

## ⚠️ Disclaimer

> **For educational purposes only.** This lab is designed for isolated virtual environments. Never run brute-force tools or attack simulations against systems you do not own or have explicit written permission to test.

---

## 👤 Author

**HARIPRAKASH**

- GitHub: [HARIPRAKASH805](https://github.com/HARIPRAKASH805)
- Project Link: [SOC Detection Lab](https://github.com/HARIPRAKASH805/SOC-project)

---

⭐ Star this repository if you found it helpful!
