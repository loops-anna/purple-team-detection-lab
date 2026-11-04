# Attack Simulation (Red Team)

**Team Red:** Hauwa · Theodora Fouda Tsilla Etobe

This folder holds the Red Team's attack script and its documentation for the purple-team Wazuh detection lab.

> **In plain English:** the Red Team played the role of an attacker group, but only inside our own private lab. They looked for open doors, tried to guess a login, and pretended to cover their tracks, so the Blue Team could prove Wazuh would catch each step.

---

## Files in this folder

| File | What it is |
|---|---|
| `attack_playbook.sh` | Red Team script that walks through the three attack stages and records each one in a log file |
| `README.md` | This file: purpose, logic, tool arguments, AI prompts and evidence |

---

## Purpose and safety

| Item | Detail |
|---|---|
| Goal | Generate the activity the Blue Team must detect with custom Wazuh rules |
| Target | Kali VM `192.168.17.129` (team-owned, isolated VirtualBox network) |
| Launched from | Ubuntu VM `192.168.17.130` |
| Safety | No public or third-party systems, non-existent username only, no real logs deleted, no malware, no real data moved |

---

## How the script works

`attack_playbook.sh` creates a folder `~/red-team-lab/` and writes a timestamped entry to `attack_simulation.log` for each stage, so the team has a record of what was simulated and when.

The **actual lab activity** for each stage was carried out with the commands in the next section. The screenshots in `evidence/` come from those commands, not from the script's log.

Run it inside the lab only:
```bash
chmod +x attack_playbook.sh
./attack_playbook.sh
```

---

## Exploit logic and tool arguments

| Stage | What it represents | Command used in the lab | Key arguments | What the Blue Team saw |
|---|---|---|---|---|
| 1. Reconnaissance | Scouting for open "doors" (ports) before attacking | `nmap -sT -sV -p 22,21,80 192.168.17.129 -oN ~/recon_results.txt` | `-sT` TCP connect scan · `-sV` detect service version · `-p` only ports 22, 21, 80 · `-oN` save results | SSH (22) open; FTP (21) and HTTP (80) closed |
| 2. Authentication attack | Repeated login attempts, the pattern of password guessing | `ssh -o NumberOfPasswordPrompts=1 -o PreferredAuthentications=password -o PubkeyAuthentication=no invaliduser@192.168.17.129` (repeated 3–5 times within 2 minutes) | One password prompt per try · force password login · skip key login · `invaliduser` does not exist | Rule 100100 (Level 5) on each failure; Rule 100101 (**Level 10**) after 3 in 120 s |
| 3. Evasion (simulated) | An attacker trying to hide by clearing logs; simulated safely with a labelled test message | `sudo logger -t wazuh-lab "WAZUH LAB: Simulated authentication log tampering activity"` | `-t wazuh-lab` tags the entry · the message text is what Rule 100102 matches | Rule 100102 (**Level 12**), MITRE T1070.002 |

---

## MITRE ATT&CK mapping

| Stage | Technique | ID |
|---|---|---|
| Reconnaissance | Network Service Discovery | T1046 |
| Authentication attack | Brute Force | T1110 |
| Evasion | Indicator Removal: Clear Linux or Mac System Logs | T1070.002 |

---

## Evidence

| Figure | File in `evidence/` | What it shows |
|---|---|---|
| 6 | `fig06-stage1-nmap-recon.png` | Stage 1: Nmap scan result |
| 7 | `fig07-stage2-ssh-login-attempts.png` | Stage 2: repeated failed SSH logins |
| 8 | `fig08-stage2-kali-journal-failed-logins.png` | Stage 2: failures recorded on Kali |
| 11 | `fig11-stage2-rule-100101-level10.png` | Stage 2: Level 10 alert raised |
| 12 | `fig12-stage3-logger-marker.png` | Stage 3: harmless marker written |
| 13 | `fig13-stage3-journal-marker.png` | Stage 3: marker recorded on Kali |
| 14 | `fig14-stage3-rule-100102-level12.png` | Stage 3: Level 12 alert raised |

---

## Limits

- Lab-only: two virtual machines on a private network.
- Stage 3 was simulated with a test message; no real logs were changed.
- The mock file-transfer part of Stage 3 was not used as evidence in the final report.
