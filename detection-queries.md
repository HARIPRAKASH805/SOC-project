# 🔍 Complete Detection Queries Library

> All queries use `sourcetype=linux_secure` to filter only SSH/auth logs and avoid false positives from other log sources.

---

## Query 1: Failed SSH Attempts by Source IP (Basic)

**Purpose:** Lists all IPs that had any failed SSH login attempts, sorted by most attempts first.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| eval src_ip=coalesce(src_ip, src)
| stats count by src_ip
| sort - count
```

---

## Query 2: Brute Force Detection — 10+ Failures in 5 Minutes

**Purpose:** Detects brute-force attacks by flagging IPs that exceed 10 failed logins within any 5-minute window.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10
| sort - count
```

---

## Query 3: High-Speed Attack Detection — 20+ Failures in 1 Minute

**Purpose:** Catches ultra-fast automated brute-force tools like Hydra running at full speed.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=1m
| stats count by src_ip, _time
| where count > 20
| sort - count
```

---

## Query 4: Username Enumeration Detection

**Purpose:** Detects attackers trying many different usernames (a sign of credential stuffing or username enumeration).

```spl
index=main sourcetype=linux_secure "Failed password for invalid user"
| rex "invalid user (?<username>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count dc(username) as unique_users by src_ip
| where unique_users > 5
| sort - unique_users
```

---

## Query 5: Successful Login AFTER Brute Force (Critical!)

**Purpose:** The most important query — detects an IP that first failed many times, then successfully logged in. This means the attacker found the correct password.

```spl
index=main sourcetype=linux_secure ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| transaction src_ip maxspan=10m
| where match(_raw, "Failed") AND match(_raw, "Accepted")
| table _time, src_ip, duration, eventcount
```

> ⚠️ **Bug fix note:** The original query used `like(events, ...)` which is invalid — `events` is not a real field. The correct field to search inside a transaction is `_raw`.

---

## Query 6: Timeline Chart — Attack Over Time

**Purpose:** Creates a visual time-series chart showing which IPs attacked and when. Use this in Splunk dashboards.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| timechart count by src_ip limit=10
```

---

## Query 7: Geographic / Port Context Enrichment

**Purpose:** Extracts more details from each failed login — username attempted, source IP, and port — for richer investigation.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "Failed password for (?:invalid user )?(?<username>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+) port (?<src_port>\d+)"
| stats count by src_ip, username, src_port
| sort - count
```

---

## Query 8: Top Targeted Usernames

**Purpose:** Shows which usernames attackers are trying the most — useful for hardening (e.g., disable the `root` account).

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "Failed password for (?:invalid user )?(?<username>\w+)"
| stats count by username
| sort - count
| head 20
```

---

## 📝 Notes for All Queries

- Always include `sourcetype=linux_secure` — without it, queries search ALL logs and produce false positives
- The `rex` command extracts `src_ip` from raw log text — if your Splunk already has field extraction configured, replace `rex` with just referencing the `src` field directly
- Adjust thresholds (`count > 10`, `count > 20`) based on your environment's normal login behavior
- For production use, save these as **Splunk Saved Searches** and attach alerts to them
