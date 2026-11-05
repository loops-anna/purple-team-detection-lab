Red Team / Blue Team Wazuh Detection Lab

Tech Mentorship Program 2026 · Cybersecurity Track (F) · Final Project

A controlled purple-team exercise in an isolated VirtualBox lab. The Red Team simulates a three-stage attack (reconnaissance → SSH password guessing → simulated log tampering). The Blue Team detects every stage with custom Wazuh rules (Level 10 and Level 12), then blocks the attacker with a UFW firewall rule.

we built a safe practice network, had half the team act as hackers, and showed that free monitoring software (Wazuh) can spot the attack, raise a high-priority alarm, and help defenders lock the attacker out.

Team
Member	Role
Nunoo Annah Frimpomaah	Team Lead · Blue Team
Thobile Mahlangu	Blue Team
Hauwa	Red Team
Theodora Fouda Tsilla Etobe	Red Team
Destiny Awambeng	Project Documentation
Results at a glance
Stage	Red Team action	Blue Team detection	Result
1. Reconnaissance	Nmap scan of ports 21, 22, 80	SSH (22) found as only open service	✅
2. Password guessing	Repeated SSH logins as a non-existent user	Rule 100100 (L5) per failure → Rule 100101 (L10) after 3 in 120 s	✅
3. Evasion (simulated)	Harmless "log tampering" marker written to journald	Rule 100102 (L12), MITRE T1070.002, fired 4 times	✅
Response	Attacker tries SSH again	UFW denies 192.168.17.130 on port 22; connection times out	✅
Lab architecture
Machine	Role	IP
Ubuntu (loop)	Wazuh 4.14.9 manager, indexer, dashboard; Red Team launch point	192.168.17.130
Kali (kali)	Protected endpoint, Wazuh agent ID 001, SSH, UFW	192.168.17.129

Private VirtualBox network 192.168.17.0/24, with no route to outside targets.

Repository layout
purple-team-detection-lab/
├── README.md                 # this file: project overview, setup, usage, limits
├── attack-simulation/        # Red Team: attack_playbook.sh + Markdown with exploit logic and AI prompts
├── detection-rules/          # Blue Team: local_rules.xml (custom rules 100100–100102)
├── response/                 # UFW firewall block and rollback commands
├── evidence/                 # Screenshots: Figures 1–18 referenced in the report
└── docs/                     # Final report (.docx), presentation (.pptx), proposal
Setup
Prerequisites
Oracle VirtualBox
Ubuntu VM (≥ 4 vCPU / 8 GB RAM recommended for Wazuh all-in-one)
Kali Linux VM
Both VMs on a host-only or internal network
1. Install Wazuh server (Ubuntu)

Follow the official Wazuh quickstart (all-in-one) for version 4.14.x, then confirm:

bash
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard --no-pager
2. Install and register the agent (Kali)

Install the Wazuh agent pointing at 192.168.17.130, then:

bash
sudo systemctl enable --now wazuh-agent
# on Ubuntu:
sudo /var/ossec/bin/agent_control -l     # expect: ID 001, kali, Active
3. Load the custom rules (Ubuntu)
bash
sudo cp detection-rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml
sudo /var/ossec/bin/wazuh-analysisd -t ; echo $?    # expect 0
sudo systemctl restart wazuh-manager
Usage: run the exercise (lab only)

Stage 1: Reconnaissance (Ubuntu)

bash
nmap -sT -sV -p 22,21,80 192.168.17.129 -oN ~/recon_results.txt

Stage 2: Controlled login attempts (Ubuntu): repeat 3–5 times within 2 minutes

bash
ssh -o NumberOfPasswordPrompts=1 -o PreferredAuthentications=password \
    -o PubkeyAuthentication=no invaliduser@192.168.17.129

Stage 3: Harmless log-tampering marker (Kali)

bash
sudo logger -t wazuh-lab "WAZUH LAB: Simulated authentication log tampering activity"

Check detections: Wazuh dashboard → Threat Hunting → Events, search: rule.id: 100100 · rule.id: 100101 · rule.id: 100102

Or test the rule offline:

bash
sudo /var/ossec/bin/wazuh-logtest     # paste the marker line → expect id 100102, level 12

Response: block the attacker (Kali)

bash
sudo ufw enable
sudo ufw deny from 192.168.17.130 to any port 22 proto tcp
sudo ufw status numbered
# verify from Ubuntu:
nc -vz -w 5 192.168.17.129 22          # expect: timed out
# roll back after testing:
sudo ufw delete deny from 192.168.17.130 to any port 22 proto tcp
Custom rules summary
Rule	Level	Detects	MITRE
100100	5	Each failed SSH password attempt	n/a
100101	10	3 failures from the same source IP within 120 s	T1110
100102	12	Simulated log-tampering marker	T1070.002
Test notes
wazuh-analysisd -t returned exit code 0 after the rule changes.
wazuh-logtest Phase 3 matched 100102 / Level 12.
Rule 100100 produced 19 hits in 24 h; Rule 100101 fired at Level 10 (3 Nov 2026, 22:49:45).
Rule 100102 fired 4 times (3 Nov 20:40, 23:28; 4 Nov 00:36, 10:46), so the result is repeatable.
Journal-to-alert timing: marker 10:46:57, then alert 10:46:58.
Port 22 was open before the UFW rule and timed out after it.
Lesson: an early version of Rule 100101 used an unsupported tag. It was fixed with <same_srcip />, and the syntax test caught it.
Limits
Two-VM lab only; it is not a full enterprise network.
Ubuntu doubled as Wazuh server and attack machine. A real attacker would be a separate host.
The firewall block was applied by the Blue Team after the alert. Automating it is the next improvement.
Log tampering was simulated with a marker. Detection of real log deletion is future work.
Wazuh raw-event archiving (logall) was off, so evidence comes from alerts and endpoint logs.
Kali's clock was not NTP-synchronised; timestamps may drift slightly.
Safety and ethics
Only team-owned VMs were tested. No public, third-party or production systems.
Login attempts used a non-existent user, with no password lists and no real accounts.
No real logs deleted, no malware, no data exfiltration.
No credentials, tokens or keys are committed to this repository.
AI use

As the Team Red brief requires, AI assistants were used to help draft attack_playbook.sh. The exact prompts are in the attack-simulation/ folder. AI also helped structure the report. All results and screenshots come from the team's own lab.
