# SOC-project
SOC lab simulating and detecting SSH brute-force attacks using Splunk SIEM | MITRE ATT&amp;CK T1110 | My implementation
# 🔐 SOC Detection Lab – SSH Brute Force Detection

> A hands-on Security Operations Center (SOC) lab for detecting SSH brute-force attacks using Splunk SIEM | MITRE ATT&CK T1110

## 🎯 Project Overview

This project demonstrates:
- **SIEM Implementation** - Splunk Enterprise for log collection
- **Attack Simulation** - SSH brute-force using Hydra
- **Detection Engineering** - SPL queries for threat hunting
- **Alert Configuration** - Automated brute force detection

## 🏗️ Architecture
┌─────────────────────────────────────┐
│ My SOC Detection Lab │
├─────────────────────────────────────┤
│ Kali Linux (Splunk + Attack) │
│ ├── Splunk Enterprise │
│ └── Hydra (SSH Brute Force) │
├─────────────────────────────────────┤
│ Ubuntu VM (Target) │
│ ├── SSH Server (port 22) │
│ └── Auth Logs (/var/log/auth.log) │
└─────────────────────────────────────┘

text

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Splunk Enterprise 9.2.0 | SIEM - Log analysis |
| Kali Linux 2024.1 | Attacker machine |
| Hydra 9.4 | Brute force tool |
| Ubuntu 22.04 LTS | Target victim |
| VirtualBox 7.0 | Virtualization |

## 🔍 Detection Query

```spl
index=main "Failed password" 
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| stats count by src_ip 
| where count > 10
