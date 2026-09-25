# Cybersecurity Home Lab

I built this to get real hands-on experience with the tools and workflows that come up in security operations roles. I'm a cybersecurity analyst and wanted to go deeper on the technical side — detection engineering, threat hunting, incident response, and web app security.

Most of this was built across two laptops with 8GB RAM each, which meant I had to be deliberate about resources and learned a lot just from making things work under those constraints.

---

## What I'm Running

**MacBook:**
- Wazuh SIEM on Ubuntu 22.04 — main detection platform
- Kali Linux — attack simulations

**Windows Laptop:**
- Ubuntu + Docker — containerized web server target
- OPNsense — firewall between the web server and the network
- Suricata — network IDS feeding alerts into Wazuh

---

## What's In This Repo

- `wazuh-siem/` — detection rules, findings, gap analysis
- `dvwa-testing/` — web app vulnerability testing notes
- `ir-runbooks/` — response documentation for attacks I've simulated
- `threat-hunting/` — hunt hypotheses, queries, results

---

## Tools

Wazuh, Suricata, Kali Linux, Hydra, Nmap, OPNsense, Docker, nginx, DVWA, Atomic Red Team
