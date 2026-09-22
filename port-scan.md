# IR Runbook — Network Reconnaissance / Port Scan
**Framework:** SANS PICERL  
**MITRE ATT&CK:** T1046 — Network Service Scanning  
**Severity:** Medium  

---

## Preparation

- Suricata IDS integrated with Wazuh for network-layer detection
- Suricata eve.json monitored via Wazuh localfile configuration
- Emerging Threats ruleset enabled in Suricata

---

## Identification

**Indicators:**
- Suricata ET SCAN alerts in Wazuh dashboard
- Single source IP hitting multiple ports in short timeframe
- Unusual volume of SYN packets without completing handshake

**Wazuh Query:**
```
rule.groups: network_scan AND data.src_ip: <source_ip>
```

**Confirmed in lab:** Nmap SYN scan (`-sS -A -p-`) from Kali detected by Suricata after integration. Initially a detection gap — host-based Wazuh alone did not catch it.

---

## Containment

1. Identify source IP and determine if internal or external
2. If external — block at perimeter firewall immediately
3. If internal — investigate the host for compromise or unauthorized tool usage

```bash
# Block at OPNsense firewall
# Firewall → Rules → WAN → Add block rule for source IP
```

---

## Eradication

1. Determine intent — authorized pen test or malicious reconnaissance
2. If malicious — review what services were discovered and exposed
3. Harden exposed services or remove unnecessary ones
4. Review firewall rules to minimize attack surface

---

## Recovery

1. Verify firewall rules are in place
2. Conduct follow-up scan from safe host to confirm reduced attack surface
3. Document services confirmed exposed during reconnaissance

---

## Lessons Learned

- **Gap found:** Wazuh without Suricata missed network scans entirely
- **Fix applied:** Installed Suricata, configured eve.json ingestion into Wazuh
- **Improvement:** Set up automated alerting threshold — single scan shouldn't page on-call, but sustained scanning should escalate
