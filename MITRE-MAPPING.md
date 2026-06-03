# 🛡️ MITRE ATT&CK Mapping

> This document maps every phase of the SSH brute-force attack simulated in this lab to the MITRE ATT&CK framework — the globally recognized standard for categorizing attacker behavior.

---

## What is MITRE ATT&CK?

MITRE ATT&CK is a free knowledge base of attacker **Tactics** (the *why* — the goal) and **Techniques** (the *how* — the method). SOC analysts use it to describe, detect, and respond to attacks in a consistent language.

- **Tactic** = The attacker's goal at this stage (e.g., "get credentials")
- **Technique** = The specific method used (e.g., "brute-force SSH")
- **Sub-technique** = A more specific variant (e.g., "password guessing")

---

## 🗺️ Full Attack Chain Mapping

### Phase 1: Initial Access via Brute Force

| Attribute         | Value                                                                 |
| ----------------- | --------------------------------------------------------------------- |
| **Tactic**        | TA0006 – Credential Access                                            |
| **Technique**     | [T1110 – Brute Force](https://attack.mitre.org/techniques/T1110/)    |
| **Sub-technique** | [T1110.001 – Password Guessing](https://attack.mitre.org/techniques/T1110/001/) |
| **Tool Used**     | Hydra 9.4                                                             |
| **Log Evidence**  | `Failed password for root from <IP> port <PORT> ssh2`                |
| **Detection**     | SPL Query 2 — 10+ failures in 5 min                                  |

**Plain English:** The attacker uses Hydra to try thousands of username/password combinations against the SSH service until one works.

---

### Phase 2: Remote Service Exploitation

| Attribute         | Value                                                                          |
| ----------------- | ------------------------------------------------------------------------------ |
| **Tactic**        | TA0008 – Lateral Movement                                                      |
| **Technique**     | [T1021.004 – Remote Services: SSH](https://attack.mitre.org/techniques/T1021/004/) |
| **Log Evidence**  | `Accepted password for <user> from <IP> port <PORT> ssh2`                     |
| **Detection**     | SPL Query 5 — Successful login after brute force                               |

**Plain English:** SSH (Secure Shell) is a legitimate remote access service. Attackers abuse it to gain remote control of the system once they have valid credentials.

---

### Phase 3: Valid Account Use (Post-Compromise)

| Attribute         | Value                                                                   |
| ----------------- | ----------------------------------------------------------------------- |
| **Tactic**        | TA0005 – Defense Evasion / TA0003 – Persistence                        |
| **Technique**     | [T1078 – Valid Accounts](https://attack.mitre.org/techniques/T1078/)   |
| **Sub-technique** | T1078.003 – Local Accounts                                              |
| **Log Evidence**  | Normal-looking SSH session after `Accepted password` event             |
| **Detection**     | Alert 2 — Brute force + successful login correlation                   |

**Plain English:** Once the attacker is logged in with a real username/password, their activity looks "normal" to basic security tools. This makes detection harder — they're not exploiting software, they're using it legitimately with stolen credentials.

---

### Phase 4: Username Enumeration (Reconnaissance)

| Attribute         | Value                                                                          |
| ----------------- | ------------------------------------------------------------------------------ |
| **Tactic**        | TA0043 – Reconnaissance                                                        |
| **Technique**     | [T1087 – Account Discovery](https://attack.mitre.org/techniques/T1087/)       |
| **Sub-technique** | T1087.001 – Local Account                                                      |
| **Log Evidence**  | `Failed password for invalid user <username> from <IP>`                       |
| **Detection**     | SPL Query 4 — Multiple unique usernames from same IP                          |

**Plain English:** When Hydra tries usernames that don't exist, the SSH server logs "invalid user." Seeing many different invalid usernames from one IP reveals the attacker is guessing which accounts exist on the system.

---

## 📊 Summary Table

| MITRE ID    | Technique Name         | Tactic              | Detected By        |
|-------------|------------------------|---------------------|--------------------|
| T1110.001   | Password Guessing      | Credential Access   | Query 2, Alert 1   |
| T1021.004   | Remote Services: SSH   | Lateral Movement    | Query 5, Alert 2   |
| T1078.003   | Local Accounts         | Defense Evasion     | Alert 2            |
| T1087.001   | Local Account Discovery| Reconnaissance      | Query 4            |

---

## 🔗 References

- [MITRE ATT&CK T1110 – Brute Force](https://attack.mitre.org/techniques/T1110/)
- [MITRE ATT&CK T1110.001 – Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [MITRE ATT&CK T1021.004 – SSH](https://attack.mitre.org/techniques/T1021/004/)
- [MITRE ATT&CK T1078 – Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [MITRE ATT&CK T1087 – Account Discovery](https://attack.mitre.org/techniques/T1087/)
