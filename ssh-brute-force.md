# IR Runbook — SSH Brute Force Attack
**Framework:** SANS PICERL  
**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing  
**Severity:** High  

---

## Preparation

- Wazuh SIEM configured to monitor `/var/log/auth.log` on all endpoints
- Rule 5760 enabled for SSH authentication failure detection
- Alert threshold: 10+ failed attempts from same IP within 60 seconds
- On-call rotation established for high severity alerts

---

## Identification

**Indicators of Compromise:**
- Multiple failed SSH authentication attempts from single IP
- Wazuh Rule 5760 firing repeatedly
- Auth log showing `Failed password` entries in rapid succession

**Wazuh Query to identify:**
```
rule.id: 5760 AND data.srcip: <source_ip>
```

**Confirmed in lab:** Hydra brute force from Kali (`192.168.1.125`) against Wazuh server (`192.168.1.9`) triggered Rule 5760 with MITRE T1110.001 mapping.

---

## Containment

**Immediate actions:**
1. Block source IP at firewall level
2. Temporarily disable SSH password authentication — enforce key-based only
3. Isolate affected system if credentials confirmed compromised

```bash
# Block attacker IP via UFW
sudo ufw deny from <attacker_ip> to any

# Disable password auth
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd
```

---

## Eradication

1. Audit all user accounts for unauthorized access
2. Review auth logs for successful logins during attack window
3. Reset credentials for any potentially compromised accounts
4. Verify no persistence mechanisms installed (cron jobs, new users, SSH keys)

```bash
# Check for new user accounts
cat /etc/passwd | grep -v nologin

# Check recently modified SSH authorized keys
find /home -name "authorized_keys" -newer /tmp/reference_time

# Review successful logins during attack window
grep "Accepted" /var/log/auth.log
```

---

## Recovery

1. Re-enable SSH with hardened configuration (key-based auth only)
2. Implement fail2ban for automatic IP blocking on future attempts
3. Verify system integrity — check for modified binaries or configs
4. Restore from clean backup if compromise confirmed

---

## Lessons Learned

- **What worked:** Wazuh detected brute force in real time with accurate MITRE mapping
- **Gap identified:** No automatic containment — response was manual. SOAR playbook needed to auto-block attacking IPs
- **Improvement:** Implement fail2ban and create automated Wazuh active response rule to block IPs after threshold breach
