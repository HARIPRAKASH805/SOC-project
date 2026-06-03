# ⚡ Alert Configuration

> All alerts use `sourcetype=linux_secure` to ensure only SSH authentication logs are evaluated, avoiding false positives from other log sources.

---

## Alert 1: SSH Brute Force Detection (Primary Alert)

### Settings

| Setting      | Value                                  |
|--------------|----------------------------------------|
| **Name**     | SSH_Brute_Force_Detection              |
| **Index**    | main                                   |
| **Sourcetype**| linux_secure                          |
| **Time Range**| Last 5 minutes                        |
| **Run Every**| 1 minute (real-time monitoring)        |
| **Trigger**  | When number of results > 0             |
| **Severity** | High                                   |

### Alert Search Query

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10
```

> ⚠️ **Bug fix note:** Original query was missing `sourcetype=linux_secure`. Without this, the alert would trigger on any log containing "Failed password" — including application logs — causing false positives.

### Response Actions

1. Log event to `threat_intel` index for tracking
2. Send email alert to SOC analyst
3. *(Optional)* Trigger automated IP block via firewall script

---

## Alert 2: Successful Login After Brute Force (Critical Alert)

**Why this matters:** This alert catches the worst-case scenario — an attacker who succeeded in guessing the password.

### Settings

| Setting      | Value                                        |
|--------------|----------------------------------------------|
| **Name**     | SSH_BruteForce_Successful_Compromise         |
| **Time Range**| Last 10 minutes                             |
| **Run Every**| 1 minute                                     |
| **Trigger**  | When number of results > 0                   |
| **Severity** | Critical                                     |

### Alert Search Query

```spl
index=main sourcetype=linux_secure ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| transaction src_ip maxspan=10m
| where match(_raw, "Failed") AND match(_raw, "Accepted")
| table _time, src_ip, duration, eventcount
```

### Response Actions

1. Immediately notify SOC team / on-call analyst
2. Log to `incident_response` index
3. Trigger automated session termination (if configured)
4. Initiate IR (Incident Response) playbook

---

## Alert 3: High-Volume Attack (Rapid Fire Detection)

**Why this matters:** Catches ultra-fast tools like Hydra running at max speed.

### Settings

| Setting      | Value                        |
|--------------|------------------------------|
| **Name**     | SSH_Rapid_BruteForce         |
| **Time Range**| Last 1 minute               |
| **Run Every**| 1 minute                     |
| **Trigger**  | When number of results > 0   |
| **Severity** | Medium                       |

### Alert Search Query

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=1m
| stats count by src_ip, _time
| where count > 20
```

---

## 📊 Alert Severity Reference

| Severity | Meaning | Example |
|----------|---------|---------|
| 🔴 Critical | Active compromise likely | Brute force + successful login |
| 🟠 High | Attack in progress | 10+ failures in 5 min |
| 🟡 Medium | Suspicious activity | 20+ failures in 1 min (fast scan) |
| 🟢 Low | Informational | Single failed login |

---

## 📝 Notes

- Save each alert as a **Splunk Scheduled Alert** under **Settings → Searches, Reports, and Alerts**
- For email alerts, configure your SMTP server under **Settings → Server Settings → Email Settings**
- Adjust thresholds based on your environment — a server with many legitimate remote users may need higher thresholds

