
**TECH MENTORSHIP PROGRAM 2026**

Cybersecurity Track • Final Project Report

**Catching an Attack in Action**

A Red Team / Blue Team detection lab using Wazuh: a controlled attack simulation, custom detection rules and firewall containment

**Fictional case study:** Asempa Community Microfinance Ltd.

**Environment:** Isolated VirtualBox lab. No real or production systems were touched.

|                                     |                                                                       |
|-------------------------------------|-----------------------------------------------------------------------|
| **Team Lead**                       | Nunoo Annah Frimpomaah                                                |
| **Red Team (attack simulation)**    | Hauwa Onize Ikunaiye • Theodora Fouda Tsilla Etobe                    |
| **Blue Team (defence & detection)** | Nunoo Annah Frimpomaah • Thobile Mahlangu                             |
| **Project Documentation**           | Destiny Awambeng                                                      |
| **Project repository**              | github.com/hauwaonizeikunaiye-max/purple-team-detection-lab (private) |
| **Date**                            | November 2026                                                         |

*Academic / training use only • All testing performed on team-owned virtual machines*

# Contents

1\. Project at a glance (plain-English summary)

2\. Executive summary

3\. The problem we addressed

4\. Scope, safety rules and limitations

5\. How the lab was built

6\. Our approach (methodology)

7\. The attack and what we detected, stage by stage

8\. Custom Wazuh detection rules

9\. Response: blocking the attacker

10\. Findings summary

11\. Detailed findings

12\. Recommendations

13\. Testing, validation and how to reproduce safely

14\. Conclusion and lessons learned

15\. Team contributions

Appendix A: Custom rules file (local_rules.xml)

Appendix B: Commands used

Appendix C: How this report meets the project requirements

Appendix D: Figure register

Appendix E: Glossary for non-technical readers

References

# 1. Project at a glance

This section explains the whole project in plain language for readers who do not work in cybersecurity.

> **In one sentence**
>
> We built a small, safe practice network, had one half of the team act as hackers, and showed that a free security-monitoring tool (Wazuh) could spot each step of the attack, raise a high-priority alarm, and help the defenders lock the attacker out.

|                    |                                                                                                                                                                                                                                                                   |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **What we built**  | Two virtual computers on one laptop: an Ubuntu computer running Wazuh (the security camera and alarm system) and a Kali Linux computer acting as the company computer being protected.                                                                            |
| **Who attacked**   | The **Red Team** played the role of a criminal group. They looked for open doors, tried to guess a password, and pretended to erase their tracks.                                                                                                                 |
| **Who defended**   | The **Blue Team** played the company's security team. They wrote their own alarm rules so Wazuh would recognise this attack, then blocked the attacker with a firewall.                                                                                           |
| **Why it matters** | Small organisations such as microfinance companies hold sensitive customer and loan data but rarely have a full-time security team. This project shows that low-cost, open-source monitoring can catch common attacks early.                                      |
| **The result**     | All three attack stages were seen by Wazuh. Our custom rules raised alerts at **Level 10** (password guessing) and **Level 12** (log tampering), both above the required Level 10. The attacker's computer was then blocked from reaching the protected computer. |

### Scorecard

| **What we set out to show**                              | **Result** | **Evidence**  |
|----------------------------------------------------------|------------|---------------|
| Wazuh installed and monitoring the protected computer    | Met        | Figures 1–4   |
| Attacker's scan of open doors (ports) recorded           | Met        | Figure 6      |
| Every failed login recorded by a custom rule (100100)    | Met        | Figures 9–10  |
| Repeated failed logins raise a Level 10 alarm (100101)   | Met        | Figure 11     |
| Log-tampering activity raises a Level 12 alarm (100102)  | Met        | Figures 13–16 |
| Attacker blocked from the protected computer by firewall | Met        | Figures 17–18 |

# 2. Executive summary

This project is a controlled Red Team / Blue Team (also called "purple team") exercise carried out in an isolated VirtualBox lab. The Red Team simulated a three-stage attack against a Linux computer: scanning for open services, repeatedly trying to log in over SSH, and simulating an attempt to tamper with system logs. The Blue Team used Wazuh, a free and open-source security monitoring platform, to collect the evidence, wrote custom detection rules, and blocked the attacking computer with a firewall rule.

The lab used an Ubuntu virtual machine (192.168.17.130) as the Wazuh server and a Kali Linux virtual machine (192.168.17.129) as the protected computer running the Wazuh agent. Both ran Wazuh version 4.14.9.

### Key outcomes

- **Visibility:** the Kali agent (ID 001) was registered and Active, and all of its security events reached the Wazuh server.

- **Reconnaissance:** a scan of ports 21, 22 and 80 showed that only SSH (port 22) was open. This became the attack path.

- **Password-guessing detection:** custom Rule 100100 recorded each failed SSH login (19 events in 24 hours), and custom Rule 100101 raised a **Level 10** alert when three failures came from the same source within 120 seconds.

- **Log-tampering detection:** custom Rule 100102 raised a **Level 12** alert, mapped to MITRE ATT&CK T1070.002, every time the simulated tampering marker was written. It fired four separate times, which shows the result can be repeated.

- **Containment:** a UFW firewall rule on Kali denied the attacking address access to SSH. A connection test from the attacker then timed out, which confirms the block worked.

### What this means

For a small financial-services organisation with no 24-hour security team, the project shows that a modest, open-source setup with well-written custom rules can turn noisy, low-priority log entries into clear, high-priority alarms that a busy IT person would notice. Our main recommendations are to make SSH key-only, to keep the custom rules, to make the firewall block happen automatically after an alert, and to protect and time-synchronise logs.

# 3. The problem we addressed

## 3.1 Case study: Asempa Community Microfinance Ltd. (fictional)

Asempa is an imaginary small microfinance company with a head office, a small operations team and a limited IT budget. It stores customer contact details, identity-verification records, loan applications and repayment records. Its IT staff manage servers remotely using SSH. There is no dedicated Security Operations Centre (SOC), so security is monitored alongside other daily work.

Management is worried that someone could quietly guess a staff password, get into an internal server, and then delete the logs that would show what they did, all without anyone noticing until the damage is done.

## 3.2 Problem statement

> **Problem statement**
>
> How can a small organisation detect suspicious remote-login activity and attempts to hide that activity, raise a clear high-priority alert, and stop the attacker, without buying an expensive enterprise security system?

## 3.3 Who this helps and why it matters

| **What is at risk**           | **What could go wrong**                    | **How monitoring helps**                                   |
|-------------------------------|--------------------------------------------|------------------------------------------------------------|
| Customer and identity records | Data theft, fraud and regulatory penalties | Spots unusual login attempts early                         |
| Loan and repayment records    | Records changed without permission         | Keeps an investigation trail                               |
| Remote administration (SSH)   | An entry point into internal systems       | Alerts on repeated failed logins and supports blocking     |
| System logs                   | Attackers erase evidence of what they did  | Alerts on tampering and keeps a copy on the central server |
| Customer trust                | Reputation damage after a breach           | Shows the organisation can detect and respond              |

**Intended users:** small-organisation IT administrators, junior SOC analysts, and managers who need to understand what security monitoring can do for them.

# 4. Scope, safety rules and limitations

## 4.1 In scope (what we tested)

- Ubuntu VM at 192.168.17.130: Wazuh manager, indexer and dashboard. This VM was also used by the Red Team to launch the simulated attack.

- Kali Linux VM at 192.168.17.129: the protected computer running the Wazuh agent (agent ID 001) and the SSH service.

- A private VirtualBox network (192.168.17.0/24) connecting only these two machines.

- Ports 21 (FTP), 22 (SSH) and 80 (HTTP) on the Kali VM.

- Wazuh custom rules, Wazuh alerts and the UFW firewall on the Kali VM.

## 4.2 Safety rules we followed

- Only our own virtual machines were tested. No public IPs, third-party systems, real companies or real customer data were involved.

- Login attempts used a deliberately non-existent username ("invaliduser"). No real accounts were attacked and no password lists were used.

- Log tampering was **simulated** with a harmless marker message. No real logs were deleted or changed.

- No real malware was used, and no real data was copied out.

- No passwords, tokens or private keys are shown in this report or the repository.

## 4.3 Limitations

- The lab has only two machines. It does not represent a full company network.

- For simplicity, the Ubuntu VM was both the Wazuh server and the attack machine. In a real setting the attacker would be a separate, untrusted machine.
- The Red Team ran each attack stage as a separate command rather than one automated attack_playbook.sh script.

- The firewall block was applied by the Blue Team after the alert was raised. Making this block happen automatically is the next improvement (see Section 12).

- Full raw-event archiving in Wazuh was switched off (logall = no), so evidence comes from alerts, the dashboard and the endpoint's own logs.

- The Kali VM's clock was not synchronised to a time server, so timestamps between machines may differ by a few seconds (see Finding F-05).

# 5. How the lab was built

The lab runs entirely inside VirtualBox on one host computer. The diagram below shows how the pieces fit together.

```text
+---------------------------------+          +-------------------------------+
| UBUNTU VM  192.168.17.130       |  attack  | KALI VM  192.168.17.129       |
| - Wazuh manager (rules/alerts)  | -------> | - SSH service (port 22)       |
| - Wazuh indexer (stores alerts) |          | - Wazuh agent 001 (sends logs)|
| - Wazuh dashboard (web view)    | <------- | - UFW firewall (blocks IPs)   |
| - Red Team attack launch point  |   logs   |                               |
+---------------------------------+          +-------------------------------+
      Private VirtualBox network 192.168.17.0/24 (no internet targets)
```

| **Machine**                | **Operating system and role**                | **IP address** | **Purpose**                                                                                              |
|----------------------------|----------------------------------------------|----------------|----------------------------------------------------------------------------------------------------------|
| Ubuntu VM (hostname: loop) | Ubuntu; Wazuh manager, indexer and dashboard | 192.168.17.130 | Collects and analyses events, runs custom rules, shows alerts. Also used to launch the simulated attack. |
| Kali VM (hostname: kali)   | Kali Linux; Wazuh agent                      | 192.168.17.129 | The protected computer. Runs SSH and sends its logs to Wazuh.                                            |

## 5.1 Evidence that the lab was working

![Figure 1: Ubuntu server IP address and Wazuh manager running](evidence/fig01-ubuntu-wazuh-manager-running.png)

**Figure 1. Ubuntu server IP address and Wazuh manager running**

***What it shows:** The Ubuntu VM has IP 192.168.17.130 and the Wazuh manager service is "active (running)". This is the central brain that applies rules and creates alerts.*

![Figure 2: Wazuh indexer and dashboard running on Ubuntu](evidence/fig02-ubuntu-indexer-dashboard-running.png)

**Figure 2. Wazuh indexer and dashboard running on Ubuntu**

***What it shows:** The indexer (stores alerts) and the dashboard (the web screen analysts use) are both "active (running)", so the full Wazuh platform is up.*

![Figure 3: Kali agent registered and Active on the Wazuh manager](evidence/fig03-kali-agent-active.png)

**Figure 3. Kali agent registered and Active on the Wazuh manager**

***What it shows:** The manager lists agent 001 "kali" as Active, which means the protected computer is connected and sending its logs.*

![Figure 4: Kali IP address and Wazuh agent running](evidence/fig04-kali-ip-agent-running.png)

**Figure 4. Kali IP address and Wazuh agent running**

***What it shows:** On the protected computer itself, the IP is 192.168.17.129 and the Wazuh agent (v4.14.9) is "active (running)", including its log collector.*

![Figure 5: Kali SSH service running and system time settings](evidence/fig05-kali-ssh-and-time-settings.png)

**Figure 5. Kali SSH service running and system time settings**

***What it shows:** SSH is running on Kali (this is the service the attacker targets). The bottom of the image also shows "System clock synchronized: no", which is recorded as Finding F-05.*

# 6. Our approach (methodology)

We followed a simple purple-team cycle. The Red Team attacks, the Blue Team watches, both teams compare notes, and the Blue Team improves its defences. Each step had to leave screenshot evidence.

1.  **Plan and agree the scope:** only our two VMs, harmless test actions, no real data.

2.  **Prepare the lab:** install Wazuh on Ubuntu, install the agent on Kali, and confirm the agent is Active.

3.  **Red Team runs the attack chain:** reconnaissance, then repeated logins, then a simulated cover-up.

4.  **Blue Team reviews the logs:** check what Kali recorded and what Wazuh alerted on with its default rules.

5.  **Engineer custom rules:** write rules in local_rules.xml, check the syntax, and test them with wazuh-logtest.

6.  **Re-run and confirm:** repeat the attack actions and confirm that the custom alerts fire at the right level.

7.  **Respond:** block the attacking address with a firewall rule and prove the block works.

8.  **Document:** record findings, evidence, recommendations and lessons learned.

## 6.1 Red Team and Blue Team activity by stage

| **Stage**              | **Red Team (attacker)**                                          | **Blue Team (defender)**                                                                     | **Evidence**  |
|------------------------|------------------------------------------------------------------|----------------------------------------------------------------------------------------------|---------------|
| 1\. Reconnaissance     | Scanned Kali ports 22, 21 and 80 to find open services           | Recorded the exposed service (SSH) as the main risk                                          | Figure 6      |
| 2\. Password guessing  | Made repeated SSH login attempts with a non-existent user        | Rule 100100 logs each failure; Rule 100101 raises a Level 10 alert after 3 failures in 120 s | Figures 7–11  |
| 3\. Cover-up (evasion) | Wrote a harmless "log tampering" marker to Kali's system journal | Rule 100102 raises a Level 12 alert mapped to MITRE T1070.002                                | Figures 12–16 |
| Response               | Attacker attempts to connect again                               | UFW rule denies the attacker's IP on port 22; connection times out                           | Figures 17–18 |

## 6.2 Tools used

| **Tool**                                          | **Used for**                                                                   |
|---------------------------------------------------|--------------------------------------------------------------------------------|
| Oracle VirtualBox                                 | Running the two isolated virtual machines                                      |
| Wazuh 4.14.9 (manager, indexer, dashboard, agent) | Collecting logs, applying rules, raising and displaying alerts                 |
| Nmap 7.94                                         | Stage 1 reconnaissance (port and service scan)                                 |
| OpenSSH client                                    | Stage 2 controlled login attempts                                              |
| logger / journalctl                               | Stage 3 harmless marker, and checking the system journal                       |
| wazuh-logtest / wazuh-analysisd -t                | Testing rules and checking configuration syntax                                |
| UFW (Uncomplicated Firewall)                      | Blocking the attacking IP address                                              |
| netcat (nc)                                       | Confirming the block by testing the connection                                 |

# 7. The attack and what we detected, stage by stage

## 7.1 Stage 1: Reconnaissance (looking for open doors)

**Plain English:** before breaking in, an attacker checks which "doors" (network ports) on a computer are open. We checked three common doors: FTP (21), SSH (22) and HTTP (80).

![Figure 6: Nmap scan of the Kali VM on ports 21, 22 and 80](evidence/fig06-stage1-nmap-recon.png)

**Figure 6. Nmap scan of the Kali VM on ports 21, 22 and 80**

***What it shows:** The scan from Ubuntu found the host up, SSH (22) open running OpenSSH 10.2p1, and FTP (21) and HTTP (80) closed. SSH is therefore the only way in.*

| **Port** | **Service** | **State** | **What it means**                                                                        |
|----------|-------------|-----------|------------------------------------------------------------------------------------------|
| 22/tcp   | SSH         | OPEN      | Remote login is possible, so this is the attack path and must be monitored and protected |
| 21/tcp   | FTP         | Closed    | No file-transfer service exposed (good)                                                  |
| 80/tcp   | HTTP        | Closed    | No web service exposed (good)                                                            |

## 7.2 Stage 2: Password guessing against SSH

**Plain English:** the attacker repeatedly tries to log in, hoping to guess a password. Each wrong guess leaves a line in the computer's log. One wrong guess is normal (people mistype). Several in a short time is a warning sign.

![Figure 7: Red Team's repeated SSH login attempts from Ubuntu](evidence/fig07-stage2-ssh-login-attempts.png)

**Figure 7. Red Team's repeated SSH login attempts from Ubuntu**

***What it shows:** Five password-only login attempts to Kali as "invaliduser". Each one is refused with "Permission denied". This creates the failed-login trail for Wazuh to detect.*

![Figure 8: Kali's own SSH log recording the failed logins](evidence/fig08-stage2-kali-journal-failed-logins.png)

**Figure 8. Kali's own SSH log recording the failed logins**

***What it shows:** Kali's journal shows "Invalid user" and "Failed password ... from 192.168.17.130". This is the raw evidence that the Wazuh agent sends to the server.*

![Figure 9: Wazuh dashboard: default alerts alongside the custom rule](evidence/fig09-stage2-dashboard-default-and-custom.png)

**Figure 9. Wazuh dashboard: default alerts alongside the custom rule**

***What it shows:** Default Wazuh rules 5710 (non-existent user) and 5503 (PAM login failed) appear at only Level 5. Our custom rule 100100 "TEAM BLUE: SSH failed password attempt recorded" also fires on each attempt.*

![Figure 10: Custom Rule 100100: every failed SSH password attempt](evidence/fig10-stage2-rule-100100-hits.png)

**Figure 10. Custom Rule 100100: every failed SSH password attempt**

***What it shows:** Searching rule.id 100100 returns 19 hits from agent "kali" in 24 hours, all Level 5. Each failed login is captured by our own rule.*

![Figure 11: Custom Rule 100101: Level 10 alert for repeated failures](evidence/fig11-stage2-rule-100101-level10.png)

**Figure 11. Custom Rule 100101: Level 10 alert for repeated failures**

***What it shows:** Rule 100101 fired at Level 10 on 3 Nov 2026 at 22:49:45 for "Three failed SSH authentication attempts". The low-level noise became one high-priority alarm.*

> **Why the custom rule matters**
>
> Default rules flagged each failed login at Level 5, which is easy to miss among hundreds of daily events (Figure 9 shows 342 events in about 10 hours). Rule 100101 links three failures from the same source address within 120 seconds and raises a single Level 10 alert, which meets the brief's "Level 10 or higher" requirement and maps to MITRE ATT&CK T1110 (Brute Force).

## 7.3 Stage 3: Evasion (simulated log tampering)

**Plain English:** after getting in, real attackers often try to delete the logs that show what they did, like wiping fingerprints. To stay safe, we did **not** delete any real logs. Instead, we wrote a clearly labelled test message into the system log that stands in for a tampering attempt, and checked that Wazuh raised an alarm.

![Figure 12: Writing the harmless log-tampering marker on Kali](evidence/fig12-stage3-logger-marker.png)

**Figure 12. Writing the harmless log-tampering marker on Kali**

***What it shows:** The logger command writes the test message "WAZUH LAB: Simulated authentication log tampering activity" into Kali's system journal. Nothing is deleted.*

![Figure 13: Kali journal confirming the marker was recorded](evidence/fig13-stage3-journal-marker.png)

**Figure 13. Kali journal confirming the marker was recorded**

***What it shows:** journalctl shows the WAZUH LAB marker with exact timestamps (e.g. 2026-11-04 10:46:57 and 11:08:54). This proves the event exists on the endpoint before Wazuh sees it.*

![Figure 14: Wazuh dashboard: Rule 100102 Level 12 alerts](evidence/fig14-stage3-rule-100102-level12.png)

**Figure 14. Wazuh dashboard: Rule 100102 Level 12 alerts**

***What it shows:** Rule 100102 "TEAM BLUE: Simulated log tampering activity detected" fired four times at Level 12 from agent "kali" (3 Nov 20:40, 23:28; 4 Nov 00:36, 10:46). The result is repeatable.*

The 10:46:57 marker in the Kali journal (Figure 13) matches the 10:46:58 Level 12 alert on the dashboard (Figure 14). This shows the complete path: the endpoint recorded the event, the agent forwarded it, and the server's custom rule raised the alert about one second later.

# 8. Custom Wazuh detection rules

The Team Blue brief requires custom rules written in local_rules.xml that flag the Red Team's activity at Level 10 or higher; default rules alone do not pass. Our rules live in /var/ossec/etc/rules/local_rules.xml on the Ubuntu Wazuh manager. The full file is in Appendix A and in the project repository.

| **Rule ID** | **Level** | **What it detects (plain English)**                                                     | **MITRE ATT&CK**                         | **Result**             |
|-------------|-----------|-----------------------------------------------------------------------------------------|------------------------------------------|------------------------|
| 100100      | 5         | Any single failed SSH password attempt. Acts as the building block for Rule 100101.     | n/a (supporting rule)                    | Met: 19 hits           |
| 100101      | 10        | Three failed SSH logins from the same IP address within 120 seconds (password guessing) | T1110 Brute Force                        | Met: Level 10 alert    |
| 100102      | 12        | The simulated log-tampering message on the endpoint                                     | T1070.002 Clear Linux or Mac System Logs | Met: 4 Level 12 alerts |

![Figure 15: The custom rules inside local_rules.xml on the Wazuh manager](evidence/fig15-local-rules-xml.png)

**Figure 15. The custom rules inside local_rules.xml on the Wazuh manager**

***What it shows:** The rules file shows rules 100100 (Level 5), 100101 (Level 10, frequency 3 in 120 s, same source IP) and 100102 (Level 12), with their descriptions and MITRE IDs.*

## 8.1 How the rules work

- **Rule 100100** builds on Wazuh's built-in SSH (sshd) rules and matches any log line containing "Failed password".

- **Rule 100101** watches for Rule 100100 firing **3 times within 120 seconds** from the **same source IP** (\<same_srcip /\>), then raises one Level 10 alert.

- **Rule 100102** matches the exact WAZUH LAB tampering message and raises a Level 12 alert, tagged as Defense Evasion.

## 8.2 Testing the rules before relying on them

We tested the rules in two ways. First, the Wazuh configuration test (wazuh-analysisd -t) returned exit code 0, which means the rules file had no syntax errors. Second, we pasted the test message into wazuh-logtest, the Wazuh tool that shows which rule a log line would trigger.

![Figure 16: wazuh-logtest confirming Rule 100102 matches](evidence/fig16-wazuh-logtest-100102.png)

**Figure 16. wazuh-logtest confirming Rule 100102 matches**

***What it shows:** Phase 3 shows the test message matched rule id 100102, level 12, MITRE T1070.002 (Defense Evasion), with "Alert to be generated". This proves the rule logic before live testing.*

# 9. Response: blocking the attacker

**Plain English:** once the alarm went off, the defenders locked the door against the attacker. They added a firewall rule on the protected computer that refuses any SSH connection from the attacker's address.

After the Level 10 alert identified 192.168.17.130 as the source of the repeated failed logins, the Blue Team applied the following UFW rule on the Kali VM:

```bash
# On Kali: deny SSH (port 22) from the attacking address
sudo ufw deny from 192.168.17.130 to any port 22 proto tcp
sudo ufw status verbose
# On Ubuntu (attacker side): confirm the block
nc -vz -w 5 192.168.17.129 22
```

![Figure 17: UFW firewall active on Kali with the deny rule](evidence/fig17-response-ufw-deny-rule.png)

**Figure 17. UFW firewall active on Kali with the deny rule**

***What it shows:** UFW is active with default deny for incoming traffic, and rule \[1\] "22/tcp DENY IN 192.168.17.130" blocks the attacker from SSH.*

![Figure 18: Attacker's connection attempt now times out](evidence/fig18-response-connection-timeout.png)

**Figure 18. Attacker's connection attempt now times out**

***What it shows:** From Ubuntu, a connection to Kali port 22 now "timed out". Before the block, the same port was open (Figure 6), so the firewall rule is working.*

> **Before and after**
>
> **Before:** port 22 on Kali was open to 192.168.17.130 (Figure 6) and accepted login attempts (Figure 7).
>
> **After:** the same connection times out (Figure 18). The Kali agent stayed Active throughout (Figure 3), so monitoring continued while the attacker was blocked.

# 10. Findings summary

Severity is rated for a small financial-services organisation like Asempa, considering how likely the issue is to be abused and how much harm it could cause.

| **ID** | **Finding**                                            | **Severity**               | **Status in lab**                         |
|--------|--------------------------------------------------------|----------------------------|-------------------------------------------|
| F-01   | SSH is open to the network and accepts password logins | Medium                     | Exposure confirmed; hardening recommended |
| F-02   | Password-guessing attempts against SSH                 | High                       | Detected by custom Level 10 rule          |
| F-03   | Attempt to tamper with system logs (simulated)         | High                       | Detected by custom Level 12 rule          |
| F-04   | Attacker contained by host firewall rule               | Positive (control working) | Block confirmed by connection test        |
| F-05   | Kali system clock not synchronised                     | Low                        | Observed; fix recommended                 |

# 11. Detailed findings

## F-01: SSH open to the network with password login allowed

|                        |                                                                                                                                                                                                                                |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Severity**           | **Medium**. An attacker still needs a valid password, but password login makes guessing possible, and SSH gives full control of a server if it succeeds.                                                                       |
| **Affected component** | Kali VM 192.168.17.129, OpenSSH 10.2p1, TCP port 22                                                                                                                                                                            |
| **Evidence**           | Figure 6: port 22 open. Figure 7: the server offers "publickey,password" authentication. Figure 5: SSH service running.                                                                                                        |
| **Safe reproduction**  | In the isolated lab only: (1) from Ubuntu run nmap -sT -sV -p 22,21,80 192.168.17.129; (2) run ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no invaliduser@192.168.17.129 and observe the password prompt. |
| **Impact**             | If a staff account has a weak password, an outsider could guess it and gain remote access to systems holding customer and loan data.                                                                                           |
| **Recommendation**     | Set PasswordAuthentication no and use SSH keys only; allow SSH only from approved admin IPs (UFW allow-list); disable direct root login; consider MFA for remote admin.                                                        |

## F-02: Password-guessing attempts against SSH

|                        |                                                                                                                                                                                |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Severity**           | **High** (as an alert). Repeated failures from one address in a short window is a strong sign of an active attack that needs a quick response.                                 |
| **Affected component** | Kali VM SSH service; Wazuh rules 100100 and 100101                                                                                                                             |
| **Evidence**           | Figures 7–8: failed logins from 192.168.17.130. Figure 9: default rules only at Level 5. Figure 10: 19 hits for 100100. Figure 11: Level 10 alert from 100101 at 22:49:45.     |
| **Safe reproduction**  | Run the controlled ssh command (F-01) three or more times within two minutes against the lab VM, then search the Wazuh dashboard for rule.id: 100101.                          |
| **Impact**             | Without the custom rule, the attack would appear only as routine Level 5 entries and could be missed, giving an attacker more time to guess a password.                        |
| **Recommendation**     | Keep Rules 100100/100101; tune the threshold for the real environment; send Level 10+ alerts to email or chat; automatically block the source IP after the alert (Section 12). |

## F-03: Attempt to tamper with system logs (simulated)

|                        |                                                                                                                                                                                                   |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Severity**           | **High**. Log tampering usually means an attacker is already inside and trying to hide. Losing logs makes investigation and legal follow-up much harder.                                          |
| **Affected component** | Kali VM system journal (journald); Wazuh rule 100102                                                                                                                                              |
| **Evidence**           | Figure 12: marker written. Figure 13: marker in journal at 10:46:57. Figure 14: four Level 12 alerts, including 10:46:58. Figure 16: logtest match.                                               |
| **Safe reproduction**  | On Kali run: logger "WAZUH LAB: Simulated authentication log tampering activity". Then search the dashboard for rule.id: 100102. No real logs are touched.                                        |
| **Impact**             | If an attacker erased local logs undetected, the organisation could lose the evidence needed to understand a breach and notify customers or regulators.                                           |
| **Recommendation**     | Keep central log copies on the Wazuh server; add file-integrity and audit monitoring on /var/log and journald settings; extend rules to detect real deletion commands; restrict who can use sudo. |

## F-04: Attacker contained by host firewall rule

|                        |                                                                                                                                                                                                                       |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Severity**           | **Positive / informational**. This confirms a defensive control works.                                                                                                                                                |
| **Affected component** | Kali VM UFW firewall                                                                                                                                                                                                  |
| **Evidence**           | Figure 17: UFW active with 22/tcp DENY IN 192.168.17.130. Figure 18: connection from Ubuntu times out (previously open, Figure 6).                                                                                    |
| **Safe reproduction**  | On Kali: sudo ufw deny from 192.168.17.130 to any port 22 proto tcp. On Ubuntu: nc -vz -w 5 192.168.17.129 22 and expect a timeout. Roll back with sudo ufw delete deny from 192.168.17.130 to any port 22 proto tcp. |
| **Impact**             | Stops the attacker's remote-login path while monitoring continues.                                                                                                                                                    |
| **Recommendation**     | Record each block (who, why, when, expiry); review blocks regularly; automate the block so it happens within seconds of a Level 10 alert.                                                                             |

## F-05: Kali system clock not synchronised

|                        |                                                                                                                               |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| **Severity**           | **Low**. This does not let an attacker in, but it weakens investigations.                                                     |
| **Affected component** | Kali VM time settings                                                                                                         |
| **Evidence**           | Figure 5: timedatectl shows "System clock synchronized: no" although the NTP service is active.                               |
| **Safe reproduction**  | On Kali run timedatectl and read the "System clock synchronized" line.                                                        |
| **Impact**             | If clocks drift, events on different machines may not line up, making it harder to build an accurate timeline of an incident. |
| **Recommendation**     | Enable and verify time sync (sudo timedatectl set-ntp true, or chrony) on all monitored machines and the Wazuh server.        |

# 12. Recommendations

| **Priority** | **Action**                                                    | **Why**                                                               | **Owner (suggested)**           |
|--------------|---------------------------------------------------------------|-----------------------------------------------------------------------|---------------------------------|
| High         | Make SSH key-only and restrict it to approved admin addresses | Removes password guessing as an attack path (F-01, F-02)              | System administrator            |
| High         | Keep and version-control custom Rules 100100–100102           | They turn low-level noise into clear high-priority alerts             | Blue Team / detection engineer  |
| High         | Automate the firewall block after a Level 10 alert            | Cuts response time from minutes to seconds, even outside office hours | Blue Team / Wazuh administrator |
| Medium       | Send Level 10+ alerts to email or a team chat channel         | A small team cannot watch the dashboard all day                       | Wazuh administrator             |
| Medium       | Add audit and file-integrity monitoring for log files         | Detects real log deletion, not just the simulated marker (F-03)       | Blue Team                       |
| Medium       | Decide on event archiving and log retention                   | Keeps raw evidence for investigations; plan storage and privacy       | IT manager                      |
| Low          | Turn on time synchronisation on all machines                  | Accurate incident timelines (F-05)                                    | System administrator            |
| Low          | Repeat this purple-team exercise every quarter                | Checks that rules still work after changes                            | Whole team                      |

# 13. Testing, validation and how to reproduce safely

## 13.1 How we checked our work

| **Check**                     | **How**                                                    | **Result**                                                                   |
|-------------------------------|------------------------------------------------------------|------------------------------------------------------------------------------|
| Wazuh platform running        | systemctl status for manager, indexer, dashboard and agent | All active (running), Figures 1–4                                            |
| Agent connected               | agent_control -l on the manager                            | kali (001) Active, Figure 3                                                  |
| Rules syntax valid            | sudo /var/ossec/bin/wazuh-analysisd -t; echo $?           | Exit code 0 (no errors)                                                      |
| Rule logic correct            | wazuh-logtest with the test message                        | Matched 100102, Level 12, Figure 16                                          |
| Rules fire on live events     | Dashboard search by rule.id                                | 100100: 19 hits; 100101: Level 10; 100102: 4 hits at Level 12                |
| Cross-check with the endpoint | Compare Kali journal timestamps with alert timestamps      | 10:46:57 journal → 10:46:58 alert; 22:52:52 failed password → 22:52:53 alert |
| Result is repeatable          | Repeat Stage 3 at different times                          | Rule 100102 fired on 4 separate occasions                                    |
| Block works                   | nc to port 22 before and after the UFW rule                | Open before (Figure 6), timed out after (Figure 18)                          |

## 13.2 What worked and what we would improve

- **Worked well:** custom correlation (Rule 100101) clearly separated an attack pattern from single mistakes; Rule 100102 was reliable and repeatable; logtest let us check rules before going live.

- **Learned the hard way:** the first version of Rule 100101 used the wrong tag for "same source IP". Running wazuh-analysisd -t caught it, and we fixed it with the supported \<same_srcip /\> element.

- **Would improve:** automate the firewall block, run the attacker from a separate third VM, enable time sync, and detect real log-deletion commands as well as the simulated marker.

## 13.3 How another reviewer can reproduce this safely

1.  Build two VMs on a **host-only or internal** VirtualBox network with no route to other networks.

2.  Install Wazuh 4.14.x (all-in-one) on Ubuntu and the Wazuh agent on Kali; confirm the agent is Active.

3.  Copy local_rules.xml from the repository to /var/ossec/etc/rules/, run wazuh-analysisd -t, and restart wazuh-manager.

4.  Follow the commands in Appendix B for Stages 1–3, using only the non-existent user and the harmless marker.

5.  Search the dashboard for rule.id 100100, 100101 and 100102 and compare with Figures 10–14.

6.  Apply and test the UFW rule (Section 9), then remove it with sudo ufw delete deny from 192.168.17.130 to any port 22 proto tcp.

# 14. Conclusion and lessons learned

This project showed, with screenshot evidence at every step, that a free, open-source tool can detect a realistic three-stage attack in a small-organisation setting. The Red Team's reconnaissance identified SSH as the only open door. Their repeated login attempts were captured by Rule 100100 and escalated to a Level 10 alert by Rule 100101. Their simulated cover-up was caught at Level 12 by Rule 100102. The Blue Team then used the alert information to block the attacking address with a firewall rule, and proved that the block worked.

For Asempa Community Microfinance, the message is simple: you do not need an expensive system to see attacks, but you do need rules written for your own risks, alerts that reach a person, and a tested plan to respond.

### Lessons learned

- Default rules see the activity, but custom rules make it **stand out**.

- Always **test rules** (analysisd -t and logtest) before trusting them in a live setting.

- Evidence must be **matched across machines**: endpoint log, then alert, then response.

- Safety first: simulate harmful actions with harmless markers so the lesson is learned without real damage.

- Clear roles (Red, Blue, Documentation) kept the work organised and the evidence complete.

# 15. Team contributions

| **Team member**             | **Role**              | **Main contributions**                                                                                                                                                                                        |
|-----------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nunoo Annah Frimpomaah      | Team Lead; Blue Team  | Coordinated the project and timeline; led the Blue Team; deployed and checked the Wazuh manager and agent; reviewed alerts and led the defensive response.                                                    |
| Thobile Mahlangu            | Blue Team             | Analysed Kali and Wazuh logs; engineered and tested custom rules 100100–100102 in local_rules.xml; applied and verified the UFW firewall block.                                                               |
| Hauwa Onize Ikunaiye        | Red Team              | Planned the three-stage attack chain; carried out reconnaissance (Nmap scan) and the controlled SSH login attempts within the lab scope; managed the GitHub repository. |
| Theodora Fouda Tsilla Etobe | Red Team              | Built the evasion-stage simulation (harmless log-tampering marker); documented the exploit logic and tool arguments in the attack-simulation README.                                                          |
| Destiny Awambeng            | Project Documentation | Collected and organised screenshot evidence; maintained the Git repository; wrote this report and the README.                                                                                                 |

> **AI-use statement**
>
> AI assistance was used to help structure and word this report. All commands, results, screenshots and configuration come from the team's own lab work.

# Appendix A: Custom rules file (local_rules.xml)

Location on the Wazuh manager: /var/ossec/etc/rules/local_rules.xml. The rules below are the ones tested in this project, transcribed from the file shown in Figure 15. The authoritative copy is in the project repository.

```xml
<group name="local,">

  <!-- ================================= -->
  <!-- TEAM BLUE CUSTOM DETECTION RULES   -->
  <!-- ================================= -->

  <!-- STAGE 2: Record failed SSH password attempts -->
  <rule id="100100" level="5">
    <if_group>sshd</if_group>
    <match>Failed password</match>
    <description>TEAM BLUE: SSH failed password attempt recorded</description>
    <group>team_blue,ssh_authentication,</group>
  </rule>

  <!-- STAGE 2: Alert after three failed attempts -->
  <rule id="100101" level="10" frequency="3" timeframe="120">
    <if_matched_sid>100100</if_matched_sid>
    <same_srcip />
    <description>TEAM BLUE: Three failed SSH authentication attempts within 120 seconds</description>
    <mitre>
      <id>T1110</id>
    </mitre>
    <group>team_blue,ssh_brute_force,authentication_failures,</group>
  </rule>

  <!-- STAGE 3: Simulated log tampering -->
  <rule id="100102" level="12">
    <match>WAZUH LAB: Simulated authentication log tampering activity</match>
    <description>TEAM BLUE: Simulated log tampering activity detected</description>
    <mitre>
      <id>T1070.002</id>
    </mitre>
    <group>team_blue,log_tampering_simulation,</group>
  </rule>

</group>
```

**Note:** *the repository copy also contains three audit-based rules (100130–100132) prepared for future auditd monitoring. They were not part of the tests in this report.*

# Appendix B: Commands used

All commands were run only inside the isolated lab.

### B1. Lab checks

```bash
# Ubuntu (Wazuh server)
ip -br addr
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard --no-pager
sudo /var/ossec/bin/agent_control -l
# Kali (protected endpoint)
ip -br addr
sudo systemctl status wazuh-agent ssh --no-pager
timedatectl
```

### B2. Stage 1: Reconnaissance (from Ubuntu)

```bash
nmap -sT -sV -p 22,21,80 192.168.17.129 -oN ~/recon_results.txt
```

### B3. Stage 2: Controlled login attempts (from Ubuntu)

```bash
# Repeat 3-5 times within two minutes; 'invaliduser' does not exist on Kali
ssh -o NumberOfPasswordPrompts=1 -o PreferredAuthentications=password \
    -o PubkeyAuthentication=no invaliduser@192.168.17.129
# Kali: confirm the failures were logged
sudo journalctl -u ssh --since "5 minutes ago" --no-pager
```

### B4. Stage 3: Harmless log-tampering marker (on Kali)

```bash
sudo logger -t wazuh-lab "WAZUH LAB: Simulated authentication log tampering activity"
sudo journalctl -t wazuh-lab -n 10 -o short-iso
```

### B5. Rule checks (on Ubuntu)

```bash
sudo /var/ossec/bin/wazuh-analysisd -t ; echo $? # expect 0
sudo /var/ossec/bin/wazuh-logtest # paste the marker line
sudo grep '"id":"100102"' /var/ossec/logs/alerts/alerts.json | tail -n 3
```

### B6. Response

```bash
# Kali
sudo ufw deny from 192.168.17.130 to any port 22 proto tcp
sudo ufw status numbered
# Ubuntu
nc -vz -w 5 192.168.17.129 22 # expect: timed out
# Kali: roll back after testing
sudo ufw delete deny from 192.168.17.130 to any port 22 proto tcp
```

# Appendix C: How this report meets the project requirements

### C1. Cybersecurity track deliverables

| **Requirement**                                                                                                                                   | **Where it is covered**   | **Status**    |
|---------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------|---------------|
| Specific security problem and clear scope (systems, lab, test assets)                                                                             | Sections 3, 4 and 5       | Met           |
| Practical security output (assessment, detection rules, configuration)                                                                            | Sections 7–9; Appendix A  | Met           |
| Structured report: cover, executive summary, scope and limitations, methodology, findings summary, detailed findings, recommendations, appendices | Whole report              | Met           |
| Each finding: title, affected component, evidence, safe reproduction, impact/severity, remediation                                                | Section 11 (F-01 to F-05) | Met           |
| Evidence (screenshots, logs, tool output) and how it was validated                                                                                | Figures 1–18; Section 13  | Met           |
| README with setup, usage, limits and test notes; repository link                                                                                  | README.md in repository   | In repository |
| Team contribution summary                                                                                                                         | Section 15                | Met           |

### C2. Team Red brief

| **Requirement**                                                | **Where it is covered**                                                                  | **Status** |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------|------------|
| Stage 1: port scan of management ports (22, 21, 80)            | Section 7.1, Figure 6                                                                    | Met        |
| Stage 2: automated credential attack on SSH | Section 7.2, Figures 7–8: repeated SSH attempts run by hand, not automated | Partly met |
| Stage 3: evasion (simulated log clearing) inside the VM only   | Section 7.3, Figures 12–13                                                               | Met        |
| attack_playbook.sh (one automated script for all three stages) | Not produced: each stage was run as a separate command (Section 7; Appendix B) | Not met |
| Markdown file: exploit logic, tool arguments, exact AI prompts | attack-simulation/README.md covers logic and tool arguments; no AI prompts, as no script was generated | Partly met |

### C3. Team Blue brief

| **Requirement**                                              | **Where it is covered**                            | **Status**    |
|--------------------------------------------------------------|----------------------------------------------------|---------------|
| Deploy the Red Team activity in the lab and analyse the logs | Sections 7 and 13                                  | Met           |
| Custom XML rules in local_rules.xml at Level 10 or higher    | Section 8; Appendix A (100101 = L10, 100102 = L12) | Met           |
| Firewall rule (iptables/ufw) dropping the attacker's IP      | Section 9, Figures 17–18                           | Met           |
| Modified local_rules.xml file                                | Appendix A and repository                          | In repository |
| Configuration scripts/commands for the firewall block        | Section 9; Appendix B6                             | Met           |

# Appendix D: Figure register

| **Fig.** | **Section** | **Machine** | **What it proves**                       |
|----------|-------------|-------------|------------------------------------------|
| 1        | 5.1         | Ubuntu      | Server IP and Wazuh manager running      |
| 2        | 5.1         | Ubuntu      | Indexer and dashboard running            |
| 3        | 5.1         | Ubuntu      | Kali agent 001 Active                    |
| 4        | 5.1         | Kali        | Endpoint IP and Wazuh agent running      |
| 5        | 5.1         | Kali        | SSH running; clock not synchronised      |
| 6        | 7.1         | Ubuntu      | Nmap: SSH open, FTP/HTTP closed          |
| 7        | 7.2         | Ubuntu      | Repeated failed SSH logins               |
| 8        | 7.2         | Kali        | Failed logins in Kali journal            |
| 9        | 7.2         | Dashboard   | Default Level 5 rules plus custom 100100 |
| 10       | 7.2         | Dashboard   | Rule 100100: 19 hits                     |
| 11       | 7.2         | Dashboard   | Rule 100101: Level 10 alert              |
| 12       | 7.3         | Kali        | Harmless marker written                  |
| 13       | 7.3         | Kali        | Marker recorded in journal               |
| 14       | 7.3         | Dashboard   | Rule 100102: four Level 12 alerts        |
| 15       | 8           | Ubuntu      | Custom rules in local_rules.xml          |
| 16       | 8.2         | Ubuntu      | wazuh-logtest match for 100102           |
| 17       | 9           | Kali        | UFW deny rule active                     |
| 18       | 9           | Ubuntu      | Connection to port 22 times out          |

# Appendix E: Glossary for non-technical readers

| **Term**             | **Simple meaning**                                                                                |
|----------------------|---------------------------------------------------------------------------------------------------|
| Red Team             | The friendly "attackers" who act like criminals to test defences                                  |
| Blue Team            | The defenders who watch for attacks and stop them                                                 |
| Purple team          | Red and Blue working together to learn and improve                                                |
| Wazuh                | Free security software that collects logs from computers and raises alarms on suspicious activity |
| Agent                | A small Wazuh program on each protected computer that sends its logs to the server                |
| Rule / custom rule   | An instruction telling Wazuh what to look for; custom rules are ones we wrote ourselves           |
| Alert level          | How serious Wazuh thinks something is (0–15); Level 10+ is high priority                          |
| SSH                  | A way for administrators to log in to a computer remotely                                         |
| Port                 | A numbered "door" on a computer that a service listens on (SSH uses 22)                           |
| Reconnaissance       | Scouting a target to find open doors before attacking                                             |
| Brute force          | Guessing passwords over and over until one works                                                  |
| Log tampering        | Deleting or changing records to hide what an attacker did                                         |
| Firewall (UFW)       | A gatekeeper that allows or blocks network connections                                            |
| MITRE ATT&CK         | A public catalogue of attacker techniques, each with an ID (e.g. T1110)                           |
| Virtual machine (VM) | A computer simulated in software, safe for testing                                                |

# References

Tech Mentorship Program 2026. (2026). Project Sprint & Program Closeout Guide: F. Cybersecurity \[Programme guidance\].

Tech Mentorship Program 2026. (2026). TEAM RED: Attack Engineering & Automation \[Project brief\].

Tech Mentorship Program 2026. (2026). TEAM BLUE: Deflection & Custom Rule Engineering \[Project brief\].

Wazuh. (n.d.). Rules syntax and custom rules. Wazuh documentation. https://documentation.wazuh.com/current/user-manual/ruleset/

Wazuh. (n.d.). wazuh-logtest. Wazuh documentation. https://documentation.wazuh.com/current/user-manual/reference/tools/wazuh-logtest.html

MITRE. (n.d.). T1110 Brute Force. MITRE ATT&CK. https://attack.mitre.org/techniques/T1110/

MITRE. (n.d.). T1070.002 Indicator Removal: Clear Linux or Mac System Logs. MITRE ATT&CK. https://attack.mitre.org/techniques/T1070/002/

National Institute of Standards and Technology. (2024). The NIST Cybersecurity Framework (CSF) 2.0. https://doi.org/10.6028/NIST.CSWP.29
