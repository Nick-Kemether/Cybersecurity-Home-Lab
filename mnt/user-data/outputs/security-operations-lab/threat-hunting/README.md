# Threat Hunting

## Overview

Threat hunting exercises conducted using Wazuh SIEM to proactively search for attacker behavior in lab environment logs. All hunts were conducted after attack simulations to validate detection coverage and identify gaps.

---

## Hunt 1 — Brute Force Credential Attack

**Hypothesis:** An attacker is attempting to brute force SSH credentials from an external host.

**Technique:** T1110.001 — Brute Force: Password Guessing

**Hunt Query (Wazuh):**
```
rule.id: 5760 AND agent.name: *
```

**Findings:**
- Multiple authentication failures from `192.168.1.125` (Kali VM)
- 300+ failed attempts in under 5 minutes
- Rule 5760 fired correctly with T1110.001 mapping
- **Detection confirmed ✅**

**Response:** Source IP identified, would block at firewall in production environment.

---

## Hunt 2 — Network Reconnaissance

**Hypothesis:** An internal or external host is conducting port scanning against lab assets.

**Technique:** T1046 — Network Service Scanning

**Initial Hunt Query:**
```
rule.groups: network_scan
```

**Initial Findings:** No results — detection gap identified. Wazuh host-based monitoring alone does not catch network scanning.

**Gap Remediation:** Installed Suricata IDS and integrated eve.json with Wazuh.

**Post-Remediation Query:**
```
rule.groups: network_scan AND data.alert.category: "Attempted Information Leak"
```

**Findings after fix:**
- Nmap SYN scan (`-sS -A -p-`) from Kali detected
- Multiple Suricata ET SCAN alerts generated
- **Detection confirmed after remediation ✅**

**Lesson:** Host-based SIEM alone leaves network-layer blind spots. Layering network IDS (Suricata) fills the gap.

---

## Hunt 3 — Web Application Exploitation

**Hypothesis:** Attacker is exploiting a public-facing web application.

**Technique:** T1190 — Exploit Public-Facing Application

**Hunt Approach:** Reviewed Docker container logs and web server access logs for:
- SQL injection patterns
- Unusual file uploads
- Command injection in request parameters

**Findings:**
- DVWA attack simulation confirmed SQL injection, XSS, command injection, and file upload exploits all leave distinct patterns in web logs
- These patterns can be used to build Wazuh rules for production web server monitoring

**Next Step:** Ingest nginx/Docker logs into Wazuh and build custom detection rules for web attack patterns.

---

## Hunting Methodology

Each hunt follows this process:

1. **Form hypothesis** — based on threat intel or known attacker behavior
2. **Identify data sources** — which logs would show this activity
3. **Query SIEM** — search for indicators
4. **Analyze results** — true positives vs noise
5. **Document gaps** — what wasn't detected and why
6. **Remediate** — add detection coverage for gaps
7. **Retest** — confirm gap is closed
