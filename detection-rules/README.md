# Detection Rules (Blue Team)

**Team Blue:** Nunoo Annah Frimpomaah (Team Lead) · Thobile Mahlangu

This folder holds the custom Wazuh detection rules the Blue Team wrote to catch the Red Team's attack.

> Wazuh's built-in rules noticed the attack but rated it low (Level 5), which is easy to miss. We wrote our own rules so the same activity raises a clear, high-priority alarm (Level 10 and Level 12).

---

## Files in this folder

| File | What it is |
|---|---|
| `local_rules.xml` | The custom Wazuh rules (100100, 100101, 100102) |
| `README.md` | This file: what each rule does, how to install it and how we tested it |

---

## The rules

| Rule ID | Level | What it detects (plain English) | How it works | MITRE ATT&CK |
|---|---|---|---|---|
| 100100 | 5 | Any single failed SSH password attempt | Builds on Wazuh's SSH (sshd) rules and matches "Failed password" | n/a (supporting rule) |
| 100101 | **10** | Password guessing: 3 failed SSH logins from the same IP within 120 seconds | Counts Rule 100100 firing 3 times in 120 s from the same source IP (`<same_srcip />`) | T1110 Brute Force |
| 100102 | **12** | Simulated log tampering on the endpoint | Matches the exact "WAZUH LAB" test message in the system journal | T1070.002 Clear Linux or Mac System Logs |

Rules 100101 and 100102 meet the brief's requirement of **Level 10 or higher**. Rule 100100 is the building block for 100101.

---

## Where the file goes

| Item | Value |
|---|---|
| Machine | Ubuntu VM `192.168.17.130` (Wazuh manager) |
| Path | `/var/ossec/etc/rules/local_rules.xml` |
| Wazuh version | 4.14.9 |

---

## How to install (on the Wazuh manager)

```bash
# 1. Copy the rules into place
sudo cp local_rules.xml /var/ossec/etc/rules/local_rules.xml

# 2. Check for syntax errors (0 means no errors)
sudo /var/ossec/bin/wazuh-analysisd -t ; echo $?

# 3. Restart the manager to load the rules
sudo systemctl restart wazuh-manager
```

---

## How we tested the rules

| Check | How | Result |
|---|---|---|
| Syntax | `wazuh-analysisd -t` | Exit code **0** (no errors) |
| Rule logic | `wazuh-logtest`, pasting the test message | Phase 3 matched **100102, Level 12**, "Alert to be generated" |
| Rule 100100 live | Dashboard search `rule.id: 100100` | **19 hits** in 24 hours from agent `kali` |
| Rule 100101 live | Dashboard search `rule.id: 100101` | **Level 10** alert, 3 Nov 2026 at 22:49:45 |
| Rule 100102 live | Dashboard search `rule.id: 100102` | **Level 12**, fired **4 times** (3 Nov 20:40, 23:28; 4 Nov 00:36, 10:46) |
| Timing cross-check | Kali journal vs Wazuh alert | Marker 10:46:57 → alert 10:46:58 |

To test Rule 100102 yourself:
```bash
sudo /var/ossec/bin/wazuh-logtest
# then paste:
WAZUH LAB: Simulated authentication log tampering activity
```

---

## Lesson learned

An early version of Rule 100101 used an unsupported tag for "same source IP". The syntax check (`wazuh-analysisd -t`) caught it, and we fixed it with the supported `<same_srcip />` element.

---

## Evidence

| Figure | File in `evidence/` | What it shows |
|---|---|---|
| 9 | `fig09-stage2-dashboard-default-and-custom.png` | Default Level 5 alerts next to custom Rule 100100 |
| 10 | `fig10-stage2-rule-100100-hits.png` | Rule 100100: 19 hits |
| 11 | `fig11-stage2-rule-100101-level10.png` | Rule 100101: Level 10 alert |
| 14 | `fig14-stage3-rule-100102-level12.png` | Rule 100102: four Level 12 alerts |
| 15 | `fig15-local-rules-xml.png` | The rules inside `local_rules.xml` on the manager |
| 16 | `fig16-wazuh-logtest-100102.png` | `wazuh-logtest` match for Rule 100102 |
