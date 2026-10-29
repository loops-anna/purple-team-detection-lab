# purple-team-detection-lab
Wazuh-based lab for detecting simulated cyberattacks and testing automated responses.

# Purple Team Detection Lab

Wazuh-based lab for detecting simulated cyberattacks and testing automated responses.

**Tech Mentorship Program 2026 — Cybersecurity Final Project**

---

# Overview

This project simulates a multi-stage attack against a Linux target and tests whether
Wazuh can detect each stage and respond automatically.

- **Red Team** runs controlled attacks from Kali Linux.
- **Blue Team** builds custom Wazuh detection rules and automated containment.
- Every attack is **re-run after a detection is built** to confirm it works.

#Lab Environment

| Component | Role |
|---|---|
| Kali Linux | Attacker machine |
| Ubuntu | Target machine (Wazuh agent installed) |
| Wazuh Manager + Dashboard | SIEM: log collection, detection, active response |

> All attacks are performed only against lab machines we own and control.

#Attack Scenario

| Stage | Technique | MITRE ATT&CK |
|---|---|---|
| 1. Reconnaissance | Port and service scan | T1046 |
| 2. Initial Access | SSH brute force | T1110 |
| 3. Privilege Escalation | Sudo abuse | T1548.003 |
| 4. Persistence | New user / SSH key / cron job | T1136, T1098.004, T1053.003 |

#Workflow

1. Red Team runs an attack stage and logs the time, source IP, and commands.
2. Blue Team checks Wazuh for alerts.
3. Blue Team writes or tunes a rule and configures an automated response.
4. Red Team re-runs the attack to validate detection and response.
5. Evidence is saved and results are recorded.

#Repository Structure

| Folder | Owner | Contents |
|---|---|---|
| `attack-simulation/` | 🔴 Red Team | Attack write-ups and commands, one file per stage |
| `detection-rules/` | 🔵 Blue Team | Custom Wazuh rules (`local_rules.xml`) and notes |
| `response/` | 🔵 Blue Team | Active response configuration and documentation |
| `evidence/` | 🟣 Both | Screenshots of attacks, alerts, and responses |
| `docs/` | 🟣 Both | Project proposal and final report |

#Results

| Stage | Detected? | Rule ID | Automated Response | Validated? |
|---|---|---|---|---|
| 1. Recon | ⏳ | — | — | ⏳ |
| 2. SSH Brute Force | ⏳ | — | — | ⏳ |
| 3. Privilege Escalation | ⏳ | — | — | ⏳ |
| 4. Persistence | ⏳ | — | — | ⏳ |

*Updated as testing progresses (⏳ pending · ✅ done · ❌ gap).*

#Team

| Name | Team | Responsibilities |
|---|---|---|
| | 🔴 Red | |
| | 🔴 Red | |
| | 🔵 Blue | |
| | 🔵 Blue | |
| | 🟣 Lead | |

#Timeline

| Dates | Milestone |
|---|---|
| Oct 30 – Nov 1 | Lab setup and Wazuh agent connected |
| Nov 2 – 5 | Attack stages, detection rules, responses |
| Nov 6 – 7 | Validation re-runs and evidence collection |
| Nov 8 | Final report and slides |
| Nov 9 – 10 | Presentation |


## Team Structure
- **Red Team:** attack-simulation/
- **Blue Team:** detection-rules/, response/
- **Shared:** docs/, evidence/
