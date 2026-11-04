# Response: Blocking the Attacker with UFW

This folder documents how the Blue Team contained the attacker after Wazuh raised the alert.

> **In plain English:** once the alarm went off, we locked the door. A firewall rule on the protected computer (Kali) refuses SSH connections from the attacker's address. We then proved the block works.

---

## When the block was applied

| Step | What happened | Evidence |
|---|---|---|
| 1. Alert | Rule **100101** raised a **Level 10** alert for repeated failed SSH logins | Figure 11 |
| 2. Identify source | The alert and Kali's journal showed the source IP **192.168.17.130** | Figures 8, 11 |
| 3. Block | The Blue Team added a UFW deny rule on Kali for that IP on port 22 | Figure 17 |
| 4. Verify | The connection from the attacker to port 22 now times out | Figure 18 |

The block was **applied manually** by the Blue Team after the alert. Making it happen automatically is listed as the next improvement.

---

## Lab details

| Item | Value |
|---|---|
| Protected computer (where the rule is applied) | Kali VM: `192.168.17.129` |
| Attacking address (blocked) | Ubuntu VM: `192.168.17.130` |
| Service protected | SSH, TCP port 22 |
| Firewall | UFW (Uncomplicated Firewall) |

---

## 1. Apply the block (run on Kali)

```bash
# Make sure UFW is on
sudo ufw enable

# Deny SSH (port 22) from the attacking address
sudo ufw deny from 192.168.17.130 to any port 22 proto tcp

# Confirm the rule is in place
sudo ufw status verbose
sudo ufw status numbered
```

**Expected output:**
```
Status: active
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     DENY IN     192.168.17.130
```

---

## 2. Check that the block works (run on Ubuntu)

```bash
nc -vz -w 5 192.168.17.129 22
```

**Expected output:**
```
nc: connect to 192.168.17.129 port 22 (tcp) timed out: Operation now in progress
```

Before the block, the same port was **open** (see the Nmap scan, Figure 6). After the block, it **times out**, so the rule is working. The Wazuh agent on Kali stays **Active** the whole time, so monitoring continues while the attacker is locked out.

---

## 3. Remove the block after testing (run on Kali)

```bash
sudo ufw delete deny from 192.168.17.130 to any port 22 proto tcp
sudo ufw status numbered      # the deny rule should no longer be listed
```

---

## Results

| Check | Before block | After block |
|---|---|---|
| Port 22 from 192.168.17.130 | Open (Figure 6) | Timed out (Figure 18) |
| UFW rule present | No | `22/tcp DENY IN 192.168.17.130` (Figure 17) |
| Wazuh agent on Kali | Active | Active (Figure 3) |

---

## Evidence

| Figure | File in `evidence/` | What it shows |
|---|---|---|
| 6 | `fig06-stage1-nmap-recon.png` | Port 22 open before the block |
| 17 | `fig17-response-ufw-deny-rule.png` | UFW active with the deny rule |
| 18 | `fig18-response-connection-timeout.png` | Connection to port 22 times out |

---

## Good practice

- Record every block: who applied it, why, when, and when it should be removed.
- Review firewall rules regularly so old blocks don't pile up.
- Run these commands only inside the isolated lab.

## Next improvement

Make the block happen **automatically** within seconds of a Level 10 alert, so the response doesn't depend on someone watching the dashboard.
