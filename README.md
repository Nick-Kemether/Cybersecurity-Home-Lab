# Security Operations Lab

A collection of hands-on security operations exercises built around a home lab running Wazuh SIEM, Kali Linux, OPNsense, and Docker. Built to practice the core disciplines of a security operations role: threat detection, incident response, threat hunting, and adversary emulation.

---

## Lab Environment

| Component | Role |
|---|---|
| Wazuh SIEM (Ubuntu 22.04) | Log ingestion, detection, alerting |
| Suricata IDS | Network-level intrusion detection |
| Kali Linux | Attack simulation and adversary emulation |
| OPNsense | Firewall and network segmentation |
| Docker + nginx | Containerized web server target |
| Atomic Red Team | MITRE ATT&CK technique emulation |

---

## Exercises

| Folder | What's Inside |
|---|---|
| `atomic-red-team/` | Adversary emulation setup and findings |
| `wazuh-detection-rules/` | Custom detection rules authored from findings |
| `ir-runbooks/` | Incident response runbooks following SANS PICERL |
| `threat-hunting/` | Threat hunt queries and methodology |

---

## Skills Demonstrated

- SIEM deployment and custom detection development
- Adversary emulation using MITRE ATT&CK framework
- Network intrusion detection with Suricata
- Incident response documentation (SANS PICERL)
- Network segmentation and firewall rule configuration
- Web application vulnerability assessment (OWASP Top 10)
- Container deployment and security

---

*Nick Kemether — Cybersecurity Analyst*
