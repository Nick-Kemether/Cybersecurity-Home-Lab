# Wazuh SIEM — Notes and Findings

## Setup

Installed Wazuh on an Ubuntu 22.04 VM using the all-in-one installer. Ran into a few issues — the indexer kept failing on startup due to RAM constraints (the VM was initially set to 3GB, needed 4GB minimum). Once I bumped the resources it stabilized.

Connected Kali Linux as a Wazuh agent so the SIEM could monitor activity from the attack machine. Getting the two VMs to communicate took longer than expected — ended up switching from NAT to bridged networking so they could reach each other across the home network.

Integrated Suricata for network-layer detection after realizing Wazuh alone missed port scans entirely. Added the Suricata eve.json as a monitored log source in ossec.conf.

---

## Detections Confirmed

**SSH Brute Force (T1110.001)**
Ran Hydra from Kali against the Wazuh server. Rule 5760 fired within seconds, correctly mapping to T1110.001. Saw the attacker IP, timestamps, and attempt count all in the dashboard.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.9
```

**Port Scan (T1046)**
Initially nothing showed up when I ran Nmap. That was the gap — host-based monitoring doesn't see network scanning. After integrating Suricata, the same scan generated alerts immediately.

```bash
nmap -sS -A -p- 192.168.1.9
```

---

## Detection Gap Found and Fixed

Wazuh without a network IDS has a blind spot for reconnaissance. An attacker can scan your entire network and you won't see it unless you have something watching at the network layer.

Fix: installed Suricata, pointed it at the right interface, added eve.json to Wazuh's monitored files, restarted the manager. Took about 30 minutes total.

---

## Custom Detection Rules

Written to tune alerting based on what I observed in the lab. Stored in `custom-rules.xml`.

Rule 100001 — fires when the same source IP triggers rule 5760 repeatedly (reduces noise from one-off failures, escalates sustained brute force)

Rule 100002 — catches Suricata ET SCAN alerts and groups them under network_scan for easier hunting

---

## What I'd Do Differently

The install process for Wazuh is solid but the indexer is resource-hungry. If I were doing this again I'd start with at least 6GB RAM on the SIEM VM. I also want to set up automated active response so Wazuh blocks attacking IPs without manual intervention — that's next.
