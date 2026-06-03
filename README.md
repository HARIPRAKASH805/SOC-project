# 🔐 SOC Detection Lab – SSH Brute Force Attack Detection

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Splunk](https://img.shields.io/badge/Splunk-9.2.0-00A1DF?logo=splunk)](https://www.splunk.com/)
[![Kali](https://img.shields.io/badge/Kali-2024.1-557C94?logo=kalilinux)](https://www.kali.org/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT&CK-T1110.001-red)](https://attack.mitre.org/techniques/T1110/001/)

> A production-ready Security Operations Center (SOC) lab that simulates and detects SSH brute-force attacks using Splunk Enterprise SIEM. Complete with detection queries, alert rules, and incident response playbook.

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Tools Used](#-tools-used)
- [Quick Start](#-quick-start)
- [Detection Queries](#-detection-queries)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [Results](#-results)
- [Screenshots](#-screenshots)
- [Author](#-author)

## 🎯 Project Overview

This SOC lab demonstrates:

| Area | Implementation |
|------|----------------|
| **SIEM Deployment** | Splunk Enterprise on Kali Linux |
| **Log Collection** | Forwarding `/var/log/auth.log` from Ubuntu |
| **Attack Simulation** | Hydra SSH brute-force (500+ attempts/min) |
| **Detection Engineering** | 5 SPL queries for threat hunting |
| **Alerting** | Automated rules with email/throttling |
| **Framework Mapping** | MITRE ATT&CK T1110.001 |

## 🏗️ Architecture

```mermaid
graph TB
    A[Kali Linux<br/>Splunk + Hydra] -->|SSH Brute Force| B[Ubuntu 22.04<br/>SSH Server]
    B -->|Forward auth.log| A
    A -->|Detection & Alerting| C[Splunk Dashboard]
