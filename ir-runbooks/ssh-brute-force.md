# SSH Brute Force — Incident Response Notes
MITRE: T1110.001 | Severity: High

---

## What triggered this

Wazuh rule 5760 firing repeatedly from the same source IP. In the lab this was Hydra running from Kali, but the detection and response steps would be the same for a real incident.

---

## How to confirm it's real

Pull the alert in Wazuh and check:
- Same source IP across all the failures?
- Failures happening in rapid succession (automated tool vs. a person mistyping)?
- Any successful logins mixed in with the failures?

That last one matters most. A brute force that didn't get in is a different situation than one that did.

Auth log on the affected host:
```bash
grep "Failed password" /var/log/auth.log | grep <source_ip>
grep "Accepted" /var/log/auth.log  # check for successful logins
```

---

## Stop the bleeding

Block the source IP immediately:
```bash
sudo ufw deny from <attacker_ip> to any
```

If there were successful logins, lock the account while you investigate:
```bash
sudo passwd -l <username>
```

---

## Figure out what happened

If there were no successful logins you're probably fine, but still check:
- Any new user accounts created recently?
- Any new SSH authorized keys added?
- Anything unusual in cron jobs?

```bash
# New accounts
grep "useradd\|adduser" /var/log/auth.log

# Authorized keys modified recently  
find /home -name "authorized_keys" -newer /var/log/auth.log

# Crontabs
crontab -l && ls /etc/cron*
```

---

## Clean up and harden

Whether or not they got in, use this as a forcing function to tighten SSH:

```bash
# Disable password auth entirely, require keys
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no
# PermitRootLogin no
sudo systemctl restart sshd
```

Install fail2ban if it isn't already running — it handles the IP blocking automatically so you don't have to:
```bash
sudo apt install fail2ban -y
```

---

## What I'd add in a real environment

The manual IP blocking step is the weak point here. In production this should be automated — Wazuh has active response capability that can block IPs when a rule fires. That's on my list to configure next in the lab.
